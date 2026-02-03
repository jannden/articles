## A Step-by-Step Guide to NodeJS Streams

_Originally published on Aug 1, 2023 [here](https://javascript.plainenglish.io/streams-in-nodejs-step-by-step-175003e855da)._

---

Streams are great for working with large amounts of data.

Whether it means reading from a file, writing to a network socket, or transforming data on-the-fly, streams make it possible to process data in small chunks.

I wrote this guide to show you the different types of streams, their capabilities, and how to use them effectively in your web applications.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*rRAKMUMU573NVMm4v-t57A.png)

### So, what is a stream?

A stream is a sequence of data that is read or written in chunks, rather than all at once.

There are four types of streams in NodeJS:

1.  Readable streams for reading data from a source, such as a file or a network socket.
2.  Writable streams for writing data to a destination, such as a file or a network socket.
3.  Duplex streams that can both read and write data.
4.  Transform streams which are duplex streams that modify or transform the data during the reading or writing process.

Here's an example of how to create a readable stream in NodeJS:

```react
// READABLE STREAM

const fs = require('fs');

const readStream = fs.createReadStream('input.txt');

readStream.on('data', (chunk) => {
  console.log(`Received ${chunk.length} bytes of data.`);
});

readStream.on('end', () => {
  console.log('No more data.');
});
```

You can see the built-in `fs` module to create a readable stream that reads from a file named "input.txt". Two event listeners are attached to it:

- one for the "data" event, which is emitted when data is available to be read,
- and one for the "end" event, which is emitted when the stream has reached the end of the data.

When the "data" event is emitted, the callback function logs the number of bytes of data received. When the "end" event is emitted, the callback function logs a message indicating that there is no more data to be read.

### Difference between Readable and Writable streams

A readable stream is a source of data, while a writable stream is a destination for data.

A readable stream can be in one of two modes: flowing or paused.

In flowing mode, data is automatically read from the stream and emitted as data events. In paused mode, the data must be read manually by calling the read() method on the stream.

A writable stream can be in one of two modes: open or closed. When the stream is in open mode, data can be written to it using the write() method. When it is in closed mode, no more data can be written to it and the end() method is usually called to signal the end of the stream.

Here's an example of how to create a writable stream in NodeJS:

```react
// WRITABLE STREAM

const fs = require('fs');

const writeStream = fs.createWriteStream('output.txt');

writeStream.on('error', (err) => {
  console.error(`Error writing to stream: ${err}`);
});

writeStream.on('finish', () => {
  console.log('Data written to file.');
});

writeStream.write('Hello, world!');
writeStream.end();
```

This writable stream writes data to a file named "output.txt". It has event listeners for the "error" and "finish" events. The `write()` method writes the string "Hello, world!" to the stream, and the `end()` method signals that the stream is finished.

### Connecting streams with Pipes

So far, we have created two separate streams --- a readable and a writable stream. They can be connected together using the `pipe()` method.

```react
const fs = require('fs');

const readStream = fs.createReadStream('input.txt');
const writeStream = fs.createWriteStream('output.txt');

readStream.pipe(writeStream);
```

Data is automatically read from the readable stream and written to the writable stream. It reduces the risk of errors that can occur if we would manually read and write data between streams. The `pipe()` method makes sure that data is not read from the source faster than it can be written to the destination.

### How to observe the data flow with PassThrough

Sometimes, you need to manipulate or inspect the data as it passes from one stream to another, but you don't actually want to change it in any way. That's when you would use a `stream.PassThrough()` instance:

```react
const fs = require('fs');
const stream = require('stream');

const readStream = fs.createReadStream('input.txt');
const writeStream = fs.createWriteStream('output.txt');
const passThroughStream = new stream.PassThrough();

readStream.pipe(passThroughStream).pipe(writeStream);

passThroughStream.on('data', (chunk) => {
  console.log(`Observed chunk: ${chunk.toString()}`);
});

passThroughStream.on('end', () => {
  console.log('Observation complete.');
});
```

By using a `PassThrough` stream, you can easily modify or transform data as it flows between readable and writable streams, without having to manually read and write data between streams.

### How to transform the data with Transform

If you wanted to modify or transform the data between the readable and writable streams, one option could be to use the built-in `stream.Transform` class in Node.js.

Here's an example that uses a `stream.Transform` instance to convert data to uppercase before writing it to a file:

```react
const fs = require('fs');
const stream = require('stream');

class UppercaseTransform extends stream.Transform {
  _transform(chunk, encoding, callback) {
    const transformedChunk = chunk.toString().toUpperCase();
    callback(null, transformedChunk);
  }
}

const readStream = fs.createReadStream('input.txt');
const writeStream = fs.createWriteStream('output.txt');
const uppercaseTransform = new UppercaseTransform();

readStream.pipe(uppercaseTransform).pipe(writeStream);

uppercaseTransform.on('data', (chunk) => {
  console.log(`Transformed chunk: ${chunk.toString()}`);
});

uppercaseTransform.on('end', () => {
  console.log('Transformation complete.');
});
```

By using a `stream.Transform` instance, you can easily modify or transform data as it flows between the readable and writable streams, without having to manually read and write data between streams or use additional buffers.

### Where does Buffer fit into all of this

A Buffer is a temporary holding area for data. It can be used together with streams to modify data.

Building on the previous example, we could use buffers to convert data to uppercase. We would achieve the same result as with `stream.Transform` without having to create a custom transform stream.

```react
const fs = require('fs');

const readStream = fs.createReadStream('input.txt');
const writeStream = fs.createWriteStream('output.txt');

readStream.on('data', (chunk) => {
  const transformedChunk = Buffer.from(chunk.toString().toUpperCase());
  console.log(`Transformed chunk: ${transformedChunk.toString()}`);
  writeStream.write(transformedChunk);
});

readStream.on('end', () => {
  writeStream.end();
  console.log('Transformation complete.');
});
```

However, keep in mind that using a `stream.Transform` instance can be more efficient and flexible if you need to perform more complex transformations on the data between streams.

### A fun way to think about streams

Streams are like a conveyor belt that carries data from one place to another. The conveyor belt can move at different speeds, and you can add or remove items from the belt at any time.

For example, imagine that you're running a factory that produces products, and you have a conveyor belt that carries raw materials from one station to the next. You could use a stream to represent the conveyor belt, and each station on the conveyor belt could represent a different stage in the processing of the raw materials.

As the raw materials move along the conveyor belt, they might undergo various transformations at each station, such as being sorted, mixed, or processed in some way. Each station on the conveyor belt could represent a different type of stream, such as a readable stream, a writable stream, or a transform stream.

TLDR: Streams help to control the flow of data and to transform it.

### Conclusion

Hope you got a good grasp on streams in this article 😊

If you found it helpful, please click the clap 👏 button.
And feel free to comment! I'd be happy to help :)
