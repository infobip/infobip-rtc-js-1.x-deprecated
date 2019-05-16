### Introduction
Infobip RTC is a JavaScript SDK which enables you to take advantage of Infobip's WebRTC platform. It gives you ability to enrich your web applications with real-time communications in minimum time, while focusing on your application's user experience and business logic. We currently support audio calls between two WebRTC users, and phone calls between WebRTC user and actual phone device.

Here you will find overview and quick guide how to connect to Infobip WebRTC platform. There is also in-depth reference documentation available.

### Prerequisites
Infobip RTC SDK requires ES6.

### First time setup
In order to use Infobip RTC, you need to have WebRTC enabled on your account, and that's it! You are ready to make WebRTC calls. Please contact your account manager to enable WebRTC.

### Getting SDK
There are few ways that you can get distribution of our SDK. We publish it as a NPM package and as a standalone JS file hosted on a CDN.

If you want to add it as a NPM dependency, run:

```
npm install infobip-rtc --save
```

And then you use it in your project like this:

```
let InfobipRTC = require('infobip-rtc');
```

or like ES6 import:

```
import {InfobipRTC} from "infobip-rtc";
```

You can include our distribution file in your JavaScript from our CDN:

```
<script src="//rtc.cdn.infobip.com/0.0.14/infobip.rtc.js"></script>
```

There is latest tag available:

```
<script src="//rtc.cdn.infobip.com/latest/infobip.rtc.js"></script>
```

### Authentication
Since Infobip RTC is just SDK, it means you are developing your own application, and you only use Infobip RTC as dependency. Your application has your own users, which we wall call subscribers throughout this guide. So, in order to use Infobip RTC, you need to register your subscribers to our platform. Credentials your subscribers use to connect to your application are irrelevant to Infobip. We only need identity with which they will use to present themselves. And when we have their identity, we can generate token that you will assign for them to use. With that token, your subscribers can connect to our platform (using Infobip RTC SDK).

In order to generate these tokens for your subscribers, you need to call our [`/webrtc/token`](https://dev.infobip.com/webrtc/generate-token) HTTP API method with proper parameters. Also, there you will authenticate yourself against Infobip platform, so we can relate subscriber's token to you. Typically, generating token occurs after your subscribers are authenticated inside your application.
In response you will receive token, that you will use to instantiate [`InfobipRTC`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC) client in your web application.

### Infobip RTC Client
After you received token via HTTP API, you are ready to instantiate [`InfobipRTC`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC) client. It can be done using these commands:

```
let token = obtainToken(); // here you call '/webrtc/token'
let options = { debug: true }
let infobipRTC = new InfobipRTC(token, options);
```

Note that this does not actually connect to Infobip WebRTC platform, it just creates new instance of [`InfobipRTC`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC). Connecting is done via [`connect`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#connect) method. Before connecting, it is useful to set-up event handlers, so you can do something when connection is set-up, when connection is lost, etc. Events are set-up via [`on`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#on) method:

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
You can call another WebRTC subscriber, if you know it's identity. It is done via [`call`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call) method:

```
let outgoingCall = infobipRTC.call('Alice');
```

As you can see, [`call`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call) method returns instance of [`OutgoingCall`](https://github.com/infobip/infobip-rtc-js/wiki/OutgoingCall) as a result. With it you can track status of your call and respond to events. Similar as for client, you can set-up event handlers, so you can do something when called subscriber answers the call, rejects it, when call is ended, etc. You set-up event handlers with this code:

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

Most important part of call is definitively media that travels across subscribers. It can be handled in `established` event, there you have remote media stream. Example how you can use it:

```
outgoingCall.on('established', function(event) {
  console.log('Alice answered call!');
  document.getElementById('remote').srcObject = event.remoteStream;
});
```

When event handlers are set-up and call is established, there are few things that you can do with actual call. One of them, of course, is to hangup. That can be done via [`hangup`](https://github.com/infobip/infobip-rtc-js/wiki/Call#hangup) method on call, and after that, both parties will receive `hangup` event upon hangup completion.

```
outgoingCall.hangup();
```

You can simulate digit press during the call, by sending DTMF codes (Dual-Tone Multi-Frequency). It is achieved via [`sendDTMF`](https://github.com/infobip/infobip-rtc-js/wiki/Call#sendDTMF) method. Valid DTMF codes are digits `0`-`9`, `*` and `#`.

```
outgoingCall.sendDTMF('*');
```

During the call, you can also mute (and unmute) your audio:

```
outgoingCall.mute(true);
```

### Receiving a call
Besides making outgoing calls, you can also receive incoming calls. In order to do that, you need to register `incoming-call` event handler of [`InfobipRTC`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC) client. There you can define behavior on incoming call. One of the most common thing to do there is to show Answer and Reject options on some UI. For purposes of this guide, let's see example that answers incoming call as soon as it arrives:

```
infobipRTC.on('incoming-call', function(incomingCall) {
  console.log('Received incoming call from: ' + incomingCall.caller.identity);
   
  incomingCall.on('established', function() {});
  incomingCall.on('hangup', function() {});
 
  incomingCall.accept(); // or incomingCall.decline();
});
```

If you are in the middle of a call, naturally, you cannot receive second call. So, if someone makes incoming call to you while you are talking, you will receive a `missed-call` event:

```
infobipRTC.on('missed-call', function (event) {
  console.log('Missed call from: ' + event.caller.identity);
});
```

### Calling phone number
It is very much similar to calling regular WebRTC user, you just use [`callPhoneNumber`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#callPhoneNumber) method instead [`call`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call). This method accepts optional second parameter, options in which you can define from parameter. It's value will display on calling phone device as Caller ID. Result of [`callPhoneNumber`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#callPhoneNumber) is also [`OutgoingCall`](https://github.com/infobip/infobip-rtc-js/wiki/OutgoingCall) that you can do everything you could when using [`call`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call) method:

```
let outgoingCall = infobipRTC.callPhoneNumber('41793026727', { from: '33755531044' });
```
