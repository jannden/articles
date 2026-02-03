## Create an Amazon Transcribe web app with JavaScript

_Originally published on Sep 20, 2023 [here](https://medium.com/@jannden/create-an-amazon-transcribe-web-app-with-javascript-a56c14b87db2)._

---

This is the simplest way to implement speech-to-text with Amazon Transcribe --- a purely front-end approach.

However, consider this approach theoretical, because entirely front-end implementation relies on exposing your AWS secret access key, which obviously you never want to do in production. If that is an issue for you, head over to the [next article in my series](https://medium.com/@jannden/create-an-amazon-transcribe-web-app-with-aws-sockets-ac2f2e4c7004) which uses a full-stack approach.

### 1\. Initialize the project

We will start a new Vanilla JavaScript project built with Webpack.

Create a new folder for your project, create a file `package.json` and fill it with this content:

```react
{
  "name": "my-app",
  "private": true,
  "scripts": {
    "start": "webpack serve --open --config webpack.config.js"
  },
  "devDependencies": {
    "webpack": "^5.88.2",
    "webpack-cli": "^5.1.4",
    "webpack-dev-server": "^4.15.1"
  },
  "dependencies": {
    "@aws-sdk/client-transcribe-streaming": "^3.651.1",
    "buffer": "^6.0.3",
    "microphone-stream": "^6.0.1",
    "process": "^0.11.10"
  }
}
```

This tells npm that we will use Webpack as the build tool and that we will need a couple of necessary libraries:

- `@aws-sdk/client-transcribe-streaming`: This is the AWS SDK for JavaScript library that provides access to the Amazon Transcribe Streaming API.
- `microphone-stream`: This is a library that provides a stream of audio data from the user's microphone.
- `buffer` and process: These are global objects in Node.js for pollyfills in Webpack.

Save the `package.json` file and run the install command:

```react
$ npm install
```

We also need to configure Webpack, so create a file `webpack.config.js` and fill it with this content:

```react
const path = require("path");
const webpack = require("webpack");

module.exports = {
  entry: "./bundle.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "/"),
  },
  resolve: {
    fallback: {
      buffer: require.resolve("buffer/"), // Polyfill buffer
      process: require.resolve("process/browser"), // Polyfill process
    },
  },
  plugins: [
    new webpack.ProvidePlugin({
      Buffer: ["buffer", "Buffer"], // Provide Buffer globally
      process: "process/browser", // Provide process globally
    }),
  ],
  devServer: {
    static: {
      directory: path.join(__dirname, "/"),
    },
    port: 3000,
  },
  mode: "development",
};
```

It just tells Webpack where to look for files and where to serve them from.

### 2\. Import libraries and define necessary variables

Time to start building the app itself. Create a file `app.js`.

We will first import the necessary libraries into the code. While at it, add the AWS access credentials as well (read [the previous article](https://medium.com/@jannden/81cad0366418) on how to get them):

```react
import {
  TranscribeStreamingClient,
  StartStreamTranscriptionCommand,
} from "@aws-sdk/client-transcribe-streaming";
import MicrophoneStream from "microphone-stream";
import { Buffer } from "buffer";

// UPDATE THIS ACCORDING TO YOUR AWS CREDENTIALS:
import { AWS_REGION, AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY } from "../aws";
```

Let's define a couple of variables before anything else:

```react
let microphoneStream = undefined;
const language = "en-US";
const SAMPLE_RATE = 44100;
let transcribeClient = undefined;
```

### 3\. Create a microphone stream

Let's get to the bottom of the application! We will create a microphone stream. This will allow you to capture audio from the user's microphone. You can use the getUserMedia method to access the user's microphone.

```react
const createMicrophoneStream = async () => {
  microphoneStream = new MicrophoneStream();
  microphoneStream.setStream(
    await window.navigator.mediaDevices.getUserMedia({
      video: false,
      audio: true,
    })
  );
};
```

### 4\. Create a Transcribe client

Now we can create a Transcribe client so that we can send the stream to Amazon. You need to provide your AWS credentials to create a Transcribe client:

```react
const createTranscribeClient = () => {
  transcribeClient = new TranscribeStreamingClient({
    region: AWS_REGION,
    credentials: {
      accessKeyId: AWS_ACCESS_KEY_ID,
      secretAccessKey: AWS_SECRET_ACCESS_KEY,
    },
  });
};
```

### 5. Encode the stream

Amazon doesn't take just any stream. It needs to be encoded in a particular way. That's why we will first prepare a function to encode PCM chunks.

For those who are interested, PCM stands for pulse-code modulation, which is the standard method used to digitally represent analog signals. You can use the MicrophoneStream library to convert the PCM chunk into raw data. Then, you can use the DataView object to convert the raw data into a buffer.

```react
const encodePCMChunk = (chunk) => {
  const input = MicrophoneStream.toRaw(chunk);
  let offset = 0;
  const buffer = new ArrayBuffer(input.length * 2);
  const view = new DataView(buffer);
  for (let i = 0; i < input.length; i++, offset += 2) {
    let s = Math.max(-1, Math.min(1, input[i]));
    view.setInt16(offset, s < 0 ? s * 0x8000 : s * 0x7fff, true);
  }
  return Buffer.from(buffer);
};
```

There is a bit more work left. We have to create a generator function that yields audio chunks in the format that Amazon Transcribe requires. It does this by iterating over the microphone input stream, checking if the length of the chunk is less than or equal to the sample rate, and if so, encoding the chunk into the required format using the `encodePCMChunk` function.

```react
const getAudioStream = async function* () {
  for await (const chunk of microphoneStream) {
    if (chunk.length <= SAMPLE_RATE) {
      yield {
        AudioEvent: {
          AudioChunk: encodePCMChunk(chunk),
        },
      };
    }
  }
};
```

### 6\. Prepare the stream

Now that we have a way to capture audio input and encode it into the correct format, we can move on to the `startStreaming` function, which is responsible for sending the audio data to Amazon Transcribe and receiving the transcription results in real time.

```react
const  startStreaming = async (language, callback) => { const command = new  StartStreamTranscriptionCommand({ LanguageCode: language, MediaEncoding: "pcm", MediaSampleRateHertz: SAMPLE_RATE, AudioStream: getAudioStream(),
  }); const data = await transcribeClient.send(command); for  await (const event of data.TranscriptResultStream) { const results = event.TranscriptEvent.Transcript.Results; if (results.length && !results[0]?.IsPartial) { const newTranscript = results[0].Alternatives[0].Transcript; console.log(newTranscript); callback(newTranscript + " ");
    }
  }
};
```

This function starts by creating a new `StartStreamTranscriptionCommand` object with the necessary parameters, including the language code, media encoding, sample rate, and audio stream. The `AudioStream` parameter is set to the `getAudioStream` generator function that we just created.

Next, the function sends the command to Amazon Transcribe and begins receiving transcription results in real time through the `TranscriptResultStream`. The function iterates over each event in the stream, extracts the transcription results from the event, and checks if the transcription is complete. If the transcription is complete, it passes the new transcript to the `callback` function provided as an argument.

### 7\. Start recording

With the `startStreaming` function in place, we can now create a `startRecording` function that calls all the necessary functions to begin real-time transcription.

```react
export const startRecording = async (callback) => {
  if (!AWS_REGION || !AWS_ACCESS_KEY_ID || !AWS_SECRET_ACCESS_KEY) {
    alert("Set AWS env variables first.");
    return false;
  }

  if (microphoneStream || transcribeClient) {
    stopRecording();
  }
  createTranscribeClient();
  createMicrophoneStream();
  await startStreaming(language, callback);
};
```

### 8\. Stop recording

Last but not least, we need to create a `stopRecording` function that stops the microphone stream if it is active.

```react
export const stopRecording = function () {
  if (microphoneStream) {
    microphoneStream.stop();
    microphoneStream.destroy();
    microphoneStream = undefined;
  }
};
```

### 9\. A simple HTML

Now that we have finished coding the main parts of the program, we can test it out. To do this, we need to create an HTML file that includes the script we have written.

Create a new file `index.html`.

Add the following code to it:

```react
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Amazon Transcribe Example</title>
  </head>
  <body>
    <button id="start">Start Recording</button>
    <button id="stop">Stop Recording</button>
    <div id="transcription"></div>
    <script src="bundle.js"></script>
  </body>
</html>
```

This code creates a simple HTML page with two buttons to start and stop recording, a div to display the transcription, and a script tag that includes our `app.js` file.

Now we need to modify our `app.js` file to add event listeners to the buttons and display the transcription in the `transcription` div.

Add the following code to the bottom of your `app.js` file:

```react
const startButton = document.getElementById("start");
const stopButton = document.getElementById("stop");
const transcriptionDiv = document.getElementById("transcription");

let transcription = "";

startButton.addEventListener("click", async () => {
  await startRecording((text) => {
    transcription += text;
    transcriptionDiv.innerHTML = transcription;
  });
});

stopButton.addEventListener("click", () => {
  stopRecording();
  transcription = "";
  transcriptionDiv.innerHTML = "";
});
```

This code adds event listeners to the *Start *and *Stop *buttons that call the `startRecording` and `stopRecording` functions respectively. The `startRecording` function also takes a callback function that updates the transcription variable and displays it in the `transcription` div.

And that's it! Save your files and let Webpack build and serve your files:

```react
$ npm run start
```

Your browser should open automatically and you should now be able to start and stop recording and see the transcription displayed on the page.

### Conclusion

This article explained how to create an Amazon Transcribe web app with JavaScript.

If you found it helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
