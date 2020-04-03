### Introduction
Infobip RTC is a JavaScript SDK which enables you to take advantage of Infobip Voice platform, giving you the ability to enrich your web applications with real-time communications in minimum time, while you focus on your application's user experience and business logic. We currently support audio calls between two web or app users, and phone calls between web or app user and actual phone device. 

Here you will find an overview and a quick guide on how to connect to Infobip Voice platform. There is also in-depth reference documentation available. 

### Prerequisites
Infobip RTC SDK requires ES6.

### First-time setup
In order to use Infobip RTC, you need to have Web and In-app Voice enabled on your account and that's it! You are ready to make web and in-app voice calls. Please contact your account manager to enable Web and In-app Voice.

### Getting SDK
There are a few ways in which you can get our SDK. We publish it as an NPM package and as a standalone JS file hosted on a CDN. 

If you want to add it as an NPM dependency, run the following:

```
npm install infobip-rtc --save
```

And then you use it in your project like this:

```
let InfobipRTC = require('infobip-rtc');
```

or as ES6 import:

```
import {InfobipRTC} from "infobip-rtc";
```

You can include our distribution file in your JavaScript from our CDN:

```
<script src="//rtc.cdn.infobip.com/0.2.7/infobip.rtc.js"></script>
```

The latest tag is also available:

```
<script src="//rtc.cdn.infobip.com/latest/infobip.rtc.js"></script>
```

### Authentication
Since Infobip RTC is an SDK, it means you develop your own application, and you only use Infobip RTC as a dependency. Your application has your own users, which we will call subscribers throughout this guide. In order to use Infobip RTC, you need to register your subscribers on our platform. The credentials your subscribers use to connect to your application are irrelevant to Infobip. We only need the identity with which they present themselves. When we have their identity, we generate a token that you assign to them to use. With that token, your subscribers can connect to our platform (using Infobip RTC SDK).

In order to generate these tokens for your subscribers, you need to call our [`/webrtc/1/token`](https://dev.infobip.com/webrtc/generate-token) HTTP API method with proper parameters. There you authenticate yourself against Infobip platform, so we can relate the subscriber's token to you. Typically, generating a token occurs after your subscribers are authenticated inside your application. In response, you receive the token, which you can then use to instantiate [`InfobipRTC`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC) client in your web application.

### Infobip RTC Client
After you have received a token via HTTP API, you are ready to instantiate the [`InfobipRTC`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC) client. You can do that using these commands:

```
let token = obtainToken(); // here you call '/webrtc/1/token'
let options = { debug: true }
let infobipRTC = new InfobipRTC(token, options);
```
Note that this doesn’t actually connect to Infobip Voice platform, it just creates a new instance of [`InfobipRTC`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC). Connection is made via the [`connect`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#connect) method. Before connecting, it is useful to set up event handlers, so you can perform something when the connection is set up, when the connection is lost, etc. Events are set up via [`on`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#on) method:

```
infobipRTC.on('connected', function(event) {
  console.log('Connected with identity: ' + event.identity);
});
infobipRTC.on('disconnected', function(event) {
  console.log('Disconnected!');
});
```

Now you are ready to connect:

```
infobipRTC.connect();
```

### Making a call
You can call another subscriber if you know their identity. It is done via the [`call`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call) method:

```
let outgoingCall = infobipRTC.call('Alice');
```

Or if you want to initiate video call: 

```
let outgoingCall = infobipRTC.call('Alice', CallOptions.builder().setVideo(true).build());
```
As you can see, the [`call`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call) method returns an instance of [`OutgoingCall`](https://github.com/infobip/infobip-rtc-js/wiki/OutgoingCall) as the result. With it you can track the status of your call and respond to events. Similar to the client, you can set up event handlers, so you can do something when the called subscriber answers the call, rejects it, the call is ended, etc. You set up event handlers with the following code:

```
outgoingCall.on('ringing', function(event) {
  console.log('Call is ringing on Alice\'s device!');
});
outgoingCall.on('established', function(event) {
  console.log('Alice answered call!');
});
outgoingCall.on('hangup', function(event) {
  console.log('Call is done! Status: ' + JSON.stringify(event.status));
});
outgoingCall.on('error', function(event) {
  console.log('Oops, something went very wrong! Message: ' + JSON.stringify(event));
});
```
The most important part of the call is definitely the media that travels between the subscribers. It can be handled in an `established` event where you have the remote media which can be streamed into your HTML page. This is an example of how you can use it:

```
<audio id="remoteAudio" autoplay />

outgoingCall.on('established', function(event) {
  console.log('Alice answered call!');
  document.getElementById('remoteAudio').srcObject = event.remoteStream;
});
```

In case of video call, you should set to your video HTML elements both the stream from your camera and the stream that you got. Example of how you can use it:

```
<video id="localVideo" autoplay />
<video id="remoteVideo" autoplay />

outgoingCall.on('established', function(event) {
  console.log('Alice answered video call!');
  document.getElementById('localVideo').srcObject = event.localStream;
  document.getElementById('remoteVideo').srcObject = event.remoteStream;
});
```
When event handlers are set up and the call is established, there are a few things that you can do with the actual call. One of them, of course, is to hang up. That can be done via the [`hangup`](https://github.com/infobip/infobip-rtc-js/wiki/Call#hangup) method on the call, and after that, both parties will receive the `hangup` event upon hang up completion.

```
outgoingCall.hangup();
```

You can simulate digit press during the call by sending DTMF codes (Dual-Tone Multi-Frequency).This is achieved via [`sendDTMF`](https://github.com/infobip/infobip-rtc-js/wiki/Call#sendDTMF) method. Valid DTMF codes are digits from `0-9`, `*` and `#`.

```
outgoingCall.sendDTMF('*');
```

During the call, you can also mute (and unmute) your audio:

```
outgoingCall.mute(true);
```

To check if the audio is muted, call the [`muted`](https://github.com/infobip/infobip-rtc-js/wiki/Call#muted) method in the following way:

```
boolean audioMuted = outgoingCall.muted();
```

Also, you can check the [`call status`](https://github.com/infobip/infobip-rtc-js/wiki/CallStatus):

```
CallStatus status = outgoingCall.status();
```

Also, you can check information such as [`duration`](https://github.com/infobip/infobip-rtc-js/wiki/Call#duration), [`start time`](https://github.com/infobip/infobip-rtc-js/wiki/Call#startTime), [`establish time`](https://github.com/infobip/infobip-rtc-js/wiki/Call#establishTime) and [`end time`](https://github.com/infobip/infobip-rtc-js/wiki/Call#endTime) by calling these methods:

```
let duration = outgoingCall.duration();
let startTime = outgoingCall.startTime();
let establishTime = outgoingCall.establishTime();
let endTime = outgoingCall.endTime();
```

### Receiving a call
Besides making outgoing calls, you can also receive incoming calls. In order to do that, you need to register the `incoming-call` event handler of [`InfobipRTC`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC) client. That is where you define the behavior of the incoming call. One of the most common things to do is to show Answer and Reject options on the UI. For the purpose of this guide, let's look at an example that answers the incoming call as soon as it arrives:

```
infobipRTC.on('incoming-call', function(incomingCallEvent) {
  const incomingCall = incomingCallEvent.incomingCall;
  console.log('Received incoming call from: ' + incomingCall.source().identity);
   
  incomingCall.on('established', function() {});
  incomingCall.on('hangup', function() {});
 
  incomingCall.accept(); // or incomingCall.decline();
});
```

### Calling phone number
It is similar to calling the regular WebRTC user, you just use the [`callPhoneNumber`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#callPhoneNumber) method instead of [`call`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call). This method accepts an optional second parameter, where you define the `from` parameter. Its value will be displayed on the calling phone device as the Caller ID. The result of the [`callPhoneNumber`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#callPhoneNumber) is the [`OutgoingCall`](https://github.com/infobip/infobip-rtc-js/wiki/OutgoingCall) with which you can do everything as when using the [`call`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call) method:

* Example of calling phone number with `from` defined: 

```
let outgoingCall = infobipRTC.callPhoneNumber('41793026727', CallPhoneNumberOptions.builder().setFrom('33712345678').build());
```

* Example of calling phone number without `from` defined: 

```
let outgoingCall = infobipRTC.callPhoneNumber('41793026727');
```

### Browser Compatibility
We support up to 5 most recent versions of these browsers (unless otherwise indicated):

|                         |                               | ![chrome](icons/chrome.png)  | ![firefox](icons/firefox.png) | ![safari](icons/safari.png)  | ![edge](icons/edge.png)      | ![opera](icons/opera.png)  | ![opera](icons/explorer.png) |
|------------------------:|------------------------------:|:----------------------------:|:-----------------------------:|:----------------------------:|:----------------------------:|:--------------------------:|:----------------------------:|
|                         |                               | **Google Chrome**            | **Mozilla Firefox**           | **Safari***                  | **Microsoft Edge**           | **Opera**                  | **Internet Explorer**        |
| **Android**             | ![android](icons/android.png) | ![chrome](icons/yes.png)     | ![chrome](icons/yes.png)      | ![chrome](icons/no.png)      | ![chrome](icons/no.png)      | ![chrome](icons/yes.png)   | ![chrome](icons/no.png)      |
| **iOS**                 | ![ios](icons/ios.png)         | ![chrome](icons/no.png)      | ![chrome](icons/no.png)       | ![chrome](icons/yes.png)     | ![chrome](icons/no.png)      | ![chrome](icons/no.png)    | ![chrome](icons/no.png)      |
| **Linux**               | ![linux](icons/linux.png)     | ![chrome](icons/yes.png)     | ![chrome](icons/yes.png)      | ![chrome](icons/no.png)      | ![chrome](icons/no.png)      | ![chrome](icons/yes.png)   | ![chrome](icons/no.png)      |
| **macOS**               | ![macOs](icons/mac.png)       | ![chrome](icons/yes.png)     | ![chrome](icons/yes.png)      | ![chrome](icons/yes.png)     | ![chrome](icons/no.png)      | ![chrome](icons/yes.png)   | ![chrome](icons/no.png)      |
| **Windows**             | ![windows](icons/windows.png) | ![chrome](icons/yes.png)     | ![chrome](icons/yes.png)      | ![chrome](icons/no.png)      | ![chrome](icons/yes.png)     | ![chrome](icons/yes.png)   | ![chrome](icons/no.png)      |

\* WebRTC support in Safari started with version 11.

> **Note**: Mobile browsers are not able to receive calls and maintain call connectivity in the background. 
> We recommend using [iOS](https://github.com/infobip/infobip-rtc-ios) and [Android](https://github.com/infobip/infobip-rtc-android) SDKs for creating mobile WebRTC Applications.
