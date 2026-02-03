## Create an Amazon Transcribe web app

_Originally published on Sep 10, 2023 [here](https://medium.com/@jannden/create-an-amazon-transcribe-web-app-3d020ec2f729)._

---

I will walk you through the process of creating a full-stack Amazon Transcribe app.

### What are we going to build?

We will build an app that facilitates a conversation between two people speaking different languages. The user can:

- Record their voice in English
- The app will transcribe the recording into text
- The text will be translated into Spanish
- The translated text will be displayed and read out loud

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*mtSKEHO3i2mTjwPUcE-DWw.jpeg)

### The contents of this series

This project is divided into these parts:

1.  [How to get AWS access keys](https://medium.com/@jannden/81cad0366418)
2.  [Create an Amazon Transcribe web app with JavaScript](https://medium.com/@jannden/a56c14b87db2) --- purely front-end implementation using the client library in JavaScript
3.  [Create an Amazon Transcribe web app with AWS pre-signed URL](https://medium.com/@jannden/ac2f2e4c7004) --- full-stack implementation using AWS sockets in NestJS
4.  [Create an Amazon Transcribe web app with NestJS](https://medium.com/@jannden/70a0987b2fc1) --- another take on a full-stack implementation using NestJS gateways

You will find the finished code in my repositories on GitHub which are divided into [front-end](https://github.com/jannden/amazon-transcribe-frontend) and [back-end](https://github.com/jannden/amazon-transcribe-backend).

The front-end will be a simple vanilla JavaScript just to illustrate how to communicate with the back-end. The back-end will be written in NestJS and it will use the [Client Transcribe Streaming](https://www.npmjs.com/package/@aws-sdk/client-transcribe-streaming) library from AWS.

Although the AWS Transcribe library is rather powerful, I couldn't find any full-stack tutorials for it. So this article will help you to figure out all the ins and outs of streaming audio from the front-end to the back-end and then streaming it from the back-end to the AWS.

### Let's start!

The next part will be about getting your AWS access credentials ready. Read it [here](https://medium.com/@jannden/how-to-get-aws-access-keys-81cad0366418).

If you find this helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
