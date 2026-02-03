## Create an Amazon Transcribe web app with NestJS

_Originally published on Oct 10, 2023 [here](https://medium.com/@jannden/create-an-amazon-transcribe-web-app-with-nestjs-70a0987b2fc1)._

---

We will create a full-stack web app that transcribes audio from the user's microphone using Amazon Transcribe, then translates it with Amazon Translate, and finally reads the translation aloud with Amazon Polly.

> I uploaded the complete [front-end](https://github.com/jannden/amazon-transcribe-frontend) and [back-end](https://github.com/jannden/amazon-transcribe-backend) repositories for this article to my GitHub.

### Using Amazon Transcribe Streaming with NestJS

Let's first create the back-end.

This guide assumes you have a basic knowledge of NestJS. If you need a refresher, read my [NestJS Crash Course](https://medium.com/@jannden/1868c443cc33) article.

We will start a new NestJS project:

```react
$ npm i -g @nestjs/cli
$ nest new amazon-transcribe
```

We will create a new module with a gateway and a service for transcribing audio:

```react
$ nest generate module local-audio
$ nest generate gateway local-audio
$ nest generate service local-audio
```

### The env variables

Create the .env file in the root directory of your project and fill it with your AWS access credentials. You can read my previous article [How to get AWS access keys](https://medium.com/@jannden/81cad0366418) if you need help obtaining them. Fill the .env file with the relevant values:

```react
AWS_REGION=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY_ID=
```

In order NestJS to be able to read the .env file, we need to update the `./src/app.module.ts` file:

```react
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { LocalAudioModule } from './local-audio/local-audio.module';

@Module({
  imports: [ConfigModule.forRoot(), LocalAudioModule],
})
export class AppModule {}
```

We should also enable CORS for our tutorial purposes. So the `./src/main.ts` file should be updated to this:

```react
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableCors();
  await app.listen(8080);
}
bootstrap();
```

Now that the basic setup is complete, let's continue with actually building the module to transcribe the audio.

### The NestJS module

No changes should be necessary for the generated module `./src/aws-signature/local-audio.module.ts`. It should look like this:

```react
import { Module } from '@nestjs/common';
import { LocalAudioService } from './local-audio.service';
import { LocalAudioGateway } from './local-audio.gateway';
import { TranslateModule } from 'src/translate/translate.module';
import { PollyModule } from 'src/polly/polly.module';

@Module({
  providers: [LocalAudioService, LocalAudioGateway],
  imports: [TranslateModule, PollyModule],
})
export class LocalAudioModule {}
```

### The NestJS gateway

We will create a NestJS gateway which is basically a socket. It will listen for three events:

- start-audio
- binary-data
- end-audio

Upon receiving the *binary-data* event, we will use a WAV writer for temporal storage. Once the audio is complete, we will use the `LocalAudioService` for transcription (we will set up the service in the next step).

```react
import {
  SubscribeMessage,
  WebSocketGateway,
  WebSocketServer,
} from '@nestjs/websockets';

import { Writer as WavWriter } from 'wav';

import { LocalAudioService } from './local-audio.service';
import { PassThrough } from 'node:stream';
import { Server } from 'node:http';

@WebSocketGateway({ cors: '*' })
export class LocalAudioGateway {
  @WebSocketServer() server: Server;
  private wavWriter: WavWriter;

  constructor(private readonly localAudioService: LocalAudioService) {}

  @SubscribeMessage('start-audio')
  async startAudio() {
    console.log(`start-audio`);

    this.wavWriter = new WavWriter({
      sampleRate: 44100,
      bitDepth: 16,
      channels: 1,
    });
  }

  @SubscribeMessage('binary-data')
  async binaryData(client: any, payload: Buffer) {
    console.log(`got ${payload ? payload.length : 0} bytes`);

    if (this.wavWriter) {
      this.wavWriter.write(payload);
    }
  }

  @SubscribeMessage('end-audio')
  async endAudio() {
    console.log(`end-audio`);

    if (this.wavWriter) {
      this.wavWriter.end();
      // Create a new PassThrough stream
      const passThroughStream = new PassThrough();

      // Pipe the Writer to the PassThrough stream
      this.wavWriter.pipe(passThroughStream);

      const transcription = await this.localAudioService.transcribeStream(
        passThroughStream,
      );
      console.log(transcription);

      this.server.emit('transcription', transcription);
    }
  }
}
```

### The NestJS service

We'll create a convenient way to transcribe audio streams using AWS Transcribe Streaming. To achieve this, we'll develop a `LocalAudioService` class, which will handle the transcription process.

```react
import { Injectable, Logger } from '@nestjs/common';
import {
  TranscribeStreamingClient,
  StartStreamTranscriptionCommand,
} from '@aws-sdk/client-transcribe-streaming';

@Injectable()
export class LocalAudioService {
  private readonly logger: Logger = new Logger('LocalAudioController');

  private readonly transcribeClient: TranscribeStreamingClient;

  constructor() {
    this.transcribeClient = new TranscribeStreamingClient({
      region: process.env.AWS_REGION,
      credentials: {
        accessKeyId: process.env.AWS_ACCESS_KEY_ID,
        secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
      },
    });
  }

  async transcribeStream(audioSource: any): Promise<string> {
    const audioStream = async function* () {
      for await (const payloadChunk of audioSource) {
        yield { AudioEvent: { AudioChunk: payloadChunk } };
      }
    };

    const command = new StartStreamTranscriptionCommand({
      LanguageCode: 'en-US',
      MediaEncoding: 'pcm',
      MediaSampleRateHertz: 44100,
      AudioStream: audioStream(),
    });

    let transcript = '';
    let response;
    try {
      response = await this.transcribeClient.send(command);
    } catch (error) {
      this.logger.error(error);
    }
    if (response?.TranscriptResultStream) {
      for await (const event of response.TranscriptResultStream) {
        if (event.TranscriptEvent) {
          const results = event.TranscriptEvent.Transcript.Results;
          if (results.length && !results[0]?.IsPartial) {
            const newTranscript = results[0].Alternatives[0].Transcript;
            transcript += newTranscript + ' ';
          }
        }
      }
    }
    return transcript;
  }
}
```

First, we import the necessary modules. Then, we make the `LocalAudioService` class injectable in order to integrate it into the gateway we built earlier.

Inside the service, we create a logger to keep track of messages and errors that might occur. In the constructor, we set up the AWS Transcribe Streaming client, initializing it with the appropriate AWS credentials and region.

The main functionality of the service lies in the `transcribeStream` method, which accepts an audio source and returns a Promise resolving to the transcribed text. So we define an asynchronous generator function called `audioStream` within the method. This function is responsible for iterating through the audio source and yielding audio chunks wrapped in an object containing an `AudioEvent`.

Next, we configure the `StartStreamTranscriptionCommand` by providing necessary parameters like language code, media encoding, and sample rate. We also pass the `audioStream` generator function as an argument.

We use a try-catch block to send the command via the `transcribeClient`. If the response is successful, it will contain a `TranscriptResultStream`. We then iterate through this stream and extract the results from the `TranscriptEvent`. For every non-partial result, we append the new transcript to our existing transcript string.

Finally, when the iteration is complete, we return the assembled transcript. This transcription is then sent back to the client through the WebSocket server, providing a seamless and real-time audio transcription service.

### Implementing Amazon Translate

Let's translate the transcribed text for the sake of learning to use Amazon (AWS) services.

We will create a new module with a service first:

```react
$ nest generate module translate
$ nest generate service translate
```

Make sure the `./translate/translate.module.ts` looks like this:

```react
import { Module } from '@nestjs/common';
import { TranslateService } from './translate.service';

@Module({
  providers: [TranslateService],
  exports: [TranslateService],
})
export class TranslateModule {}
```

And now let's build the `./translate/translate.service.ts`:

```react
import { Injectable } from '@nestjs/common';
import {
  TranslateClient,
  TranslateTextCommand,
} from '@aws-sdk/client-translate';

@Injectable()
export class TranslateService {
  private readonly translateClient: TranslateClient;

  constructor() {
    this.translateClient = new TranslateClient({
      region: process.env.AWS_REGION,
      credentials: {
        accessKeyId: process.env.AWS_ACCESS_KEY_ID,
        secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
      },
    });
  }

  async translateText(text: string): Promise<string> {
    const command = new TranslateTextCommand({
      SourceLanguageCode: 'en',
      TargetLanguageCode: 'es',
      Text: text,
    });

    const response = await this.translateClient.send(command);
    return response.TranslatedText || '';
  }
}
```

We initialize the AWS Translate client in the constructor with the same AWS credentials and region as in the `LocalAudio` service.

The important part here is the `translateText` method, which accepts a text string as input and returns a Promise resolving to the translated text. It's configured with the `TranslateTextCommand`, which is sent to the `translateClient`. If the response is successful, we extract the translated text and return it.

Now let's use this service in the LocalAudio gateway. In order to do so, let's replace this line at the end of `./local-audio/local-audio.gateway.ts`:

```react
this.server.emit('transcription', transcription);
```

... with something like this:

```react
if (transcription) {
  const translation = await this.translateService.translateText(
    transcription,
  );
  this.server.emit('translation', translation);
}
```

### Implementing Amazon Polly

Let's not end there. Why not to pronounce what we had translated? We can use Amazon Polly for adding text-to-speech functionality.

We will create a new module with a service first:

```react
$ nest generate module polly
$ nest generate service polly
```

Make sure the `./polly/polly.module.ts` looks like this:

```react
import { Module } from '@nestjs/common';
import { PollyService } from './polly.service';

@Module({
  providers: [PollyService],
  exports: [PollyService],
})
export class PollyModule {}
```

Now building the service shouldn't feel difficult. Here is how the `./polly/polly.service.ts` could look like:

```react
import { Injectable } from '@nestjs/common';
import { Polly } from '@aws-sdk/client-polly';
import { Readable } from 'node:stream';

@Injectable()
export class PollyService {
  private readonly pollyClient: Polly;

  constructor() {
    this.pollyClient = new Polly({
      region: process.env.AWS_REGION,
      credentials: {
        accessKeyId: process.env.AWS_ACCESS_KEY_ID,
        secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
      },
    });
  }

  async generateSpeech(text: string): Promise<Readable> {
    const response = await this.pollyClient.synthesizeSpeech({
      Engine: 'standard',
      LanguageCode: 'es-ES',
      Text: text,
      OutputFormat: 'mp3',
      VoiceId: 'Miguel',
    });

    if (response.AudioStream instanceof Readable) {
      return response.AudioStream;
    }
    return null;
  }
}
```

The main functionality lies in the `generateSpeech` method, which accepts a text string as input and returns a Promise resolving to a Readable stream containing the synthesized speech.

Inside, we send a `synthesizeSpeech` request to the `pollyClient`, providing necessary parameters like engine, language code, text, output format, and voice ID. If the response is successful, we extract the `AudioStream` from the response object, check if it's an instance of `Readable`, and return it.

Let's connect this to the gateway now. So replace the following line from `./local-audio/local-audio.gateway.ts`:

```react
this.server.emit('translation', translation);
```

... with this:

```react
const audioStream = await this.pollyService.generateSpeech(translation);

audioStream.on('data', (chunk) => {
  this.server.emit('binary-data', chunk);
});

audioStream.on('end', () => {
  this.server.emit('end-transcript', true);
});
```

Now we have a pretty nice NestJS back-end that takes an audio stream and uses Amazon services to transcribe, translate and read out loud.

You can run the server with this simple command:

```react
$ npm run start
```

### Front-end implementation

Front-end would be responsible for capturing audio from the user's microphone, sending it to a WebSocket server, and then playing back the translated and synthesized audio received from the server.

This is what a simple JavaScript file could look like:

```react
import MicrophoneStream from "microphone-stream";
import { Buffer } from "buffer";
import { io } from "socket.io-client";

// UPDATE THIS ACCORDING TO YOUR BACKEND:
const ws = "ws://localhost:8080";

let microphoneStream = undefined;
let socket = undefined;

const audioElement = document.createElement("audio");
let mediaSource;
document.body.appendChild(audioElement);
let sourceBuffer;
let receivedDataLength = 0;
let pendingChunks = [];
let sourceBuffersUpdating = 0;

const encodePCMChunk = (chunk) => {
  const input = MicrophoneStream.toRaw(chunk);
  var offset = 0;
  var buffer = new ArrayBuffer(input.length * 2);
  var view = new DataView(buffer);
  for (var i = 0; i < input.length; i++, offset += 2) {
    var s = Math.max(-1, Math.min(1, input[i]));
    view.setInt16(offset, s < 0 ? s * 0x8000 : s * 0x7fff, true);
  }
  return Buffer.from(buffer);
};

const createMicrophoneStream = async () => {
  microphoneStream = new MicrophoneStream();
  microphoneStream.setStream(
    await window.navigator.mediaDevices.getUserMedia({
      video: false,
      audio: true,
    })
  );
};

export const startRecording = async (callback) => {
  mediaSource = new MediaSource();
  audioElement.src = URL.createObjectURL(mediaSource);

  if (microphoneStream) {
    stopRecording();
  }

  socket = io(ws);

  socket.on("connect", function () {
    sourceBuffer = mediaSource.addSourceBuffer("audio/mpeg");
    sourceBuffer.addEventListener("updateend", () => {
      if (pendingChunks.length > 0 && !sourceBuffer.updating) {
        const buffer = pendingChunks.shift();
        sourceBuffer.appendBuffer(buffer);
      }
      sourceBuffersUpdating--;
      if (sourceBuffersUpdating === 0 && !mediaSource.updating) {
        mediaSource.endOfStream();
        audioElement.play();
      }
    });
    if (microphoneStream && socket.connected) {
      socket.emit("start-audio", true);
      console.log("start-audio");
      microphoneStream.on("data", function (chunk) {
        socket.emit("binary-data", encodePCMChunk(chunk));
      });
    }
  });

  socket.on("transcript", function (data) {
    console.log("transcript", data);
    callback(data);
  });

  socket.on("translation", function (data) {
    console.log("translation", data);
    callback(data);
  });

  socket.on("binary-data", (chunk) => {
    console.log(`Received ${receivedDataLength} bytes`);
    receivedDataLength += chunk.byteLength;
    sourceBuffersUpdating++;
    pendingChunks.push(chunk);
    if (!sourceBuffer.updating) {
      const buffer = pendingChunks.shift();
      sourceBuffer.appendBuffer(buffer);
    }
  });

  socket.on("end-transcript", () => {
    console.log("Transcript complete");
    socket.disconnect();
  });

  createMicrophoneStream();
};

export const stopRecording = function () {
  if (microphoneStream) {
    microphoneStream.stop();
    microphoneStream.destroy();
    microphoneStream = undefined;
    socket.emit("end-audio", true);
    console.log("end-audio");
  }
};
```

First, we import a few libraries: `MicrophoneStream` helps us get the audio data from your microphone in real time; `Buffer` is useful for working with binary data efficiently; and `socket.io-client` enables us to connect and interact with a WebSocket server.

The `encodePCMChunk` function is there to take care of converting the audio chunk from the microphone stream into a PCM format. This format is what we'll send to the WebSocket server for processing (it's required by Amazon).

We create a new `MicrophoneStream` object using the `createMicrophoneStream` function, which sets up the audio stream from your microphone.

The `startRecording` function is the main entry point for starting the recording process. When called, it sets up a WebSocket connection to the server and initializes the `MicrophoneStream`. It also creates an HTML5 `audio` element to play back the synthesized audio received from the server.

When the WebSocket connection is established, we emit a "start-audio" event and start capturing data from the microphone. The microphone stream's "data" event is triggered every time there's new audio data available. Inside this event, we convert the audio chunk to PCM format using `encodePCMChunk` and send it to the server as binary data.

The server responds with various events like "transcript", "translation", and "binary-data". We listen for these events and handle them accordingly. When we receive the "binary-data" event, we append the received audio chunk to the `audio` element's source buffer. Once the "end-transcript" event is received, we disconnect from the WebSocket server, indicating the process is complete.

The `stopRecording` function stops the microphone stream, disconnects from the WebSocket server, and emits an "end-audio" event to let the server know that we've finished recording.

In summary, this JavaScript code captures audio from your microphone, sends it to a WebSocket server for processing, and plays back the translated and synthesized audio received from the server.

### Conclusion

This article showed how to create an Amazon Transcribe web app with NestJS, including also Amazon Translate and Amazon Polly.

On GitHub, you can find my complete [front-end](https://github.com/jannden/amazon-transcribe-frontend) and [back-end](https://github.com/jannden/amazon-transcribe-backend) repositories for this project.

If you found it helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
