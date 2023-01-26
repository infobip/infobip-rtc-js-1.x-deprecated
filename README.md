### Introduction

Infobip RTC is a JavaScript SDK which enables you to take advantage of Infobip platform,
giving you the ability to enrich your web applications with real-time communications in minimum time,
while you focus on your application's user experience and business logic.
We currently support audio and video calls between two web or app users,
and phone calls between a web or an app user and actual phone device.

Here you will find an overview, and a quick guide on how to connect to Infobip platform.
There is also in-depth reference documentation available.

### Prerequisites

Infobip RTC SDK requires ES6. Also secure connection over HTTPS is required, except for localhost address.

### First-time setup

In order to use Infobip RTC, you need to have Web and In-app Calls enabled on your account and that's it!
You are ready to make Web and In-app calls. To learn how to enable them
see [the documentation](https://www.infobip.com/docs/voice-and-video/webrtc#set-up-web-and-in-app-calls).

### Getting SDK

There are a few ways in which you can get our SDK.
We publish it as an NPM package and as a standalone JS file hosted on a CDN.

If you want to add it as an NPM dependency, run the following:

```bash
npm install infobip-rtc --save
```

After which you would use it in your project like this:

```javascript
let InfobipRTC = require('infobip-rtc');
```

or as ES6 import:

```javascript
import {InfobipRTC} from "infobip-rtc";
```

You can include our distribution file in your JavaScript from our CDN:

```html
<script src="//rtc.cdn.infobip.com/1.14.9/infobip.rtc.js"></script>
```

The latest tag is also available:

```html
<script src="//rtc.cdn.infobip.com/latest/infobip.rtc.js"></script>
```

### Authentication

Since Infobip RTC is an SDK, it means you develop your own application, and you only use Infobip RTC as a dependency.
Your application has your own users, which we will call subscribers throughout this guide.
So, in order to use Infobip RTC, you need to register your subscribers on our platform.
The credentials your subscribers use to connect to your application are irrelevant to Infobip.
We only need the identity they will use to present themselves. When we have the subscriber's identity,
we can generate a token assigned to that specific subscriber.
With that token, your subscribers can connect to our platform (using Infobip RTC SDK).

To generate these tokens for your subscribers, you need to call our
[`/webrtc/1/token`](https://dev.infobip.com/webrtc/generate-token) HTTP API method using proper parameters.
There you authenticate yourself against Infobip platform, so we can relate the subscriber's token to you.
Typically, generating a token occurs after your subscribers are authenticated inside your application.
You will receive the token in a response that you will use to instantiate
[`InfobipRTC`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC) client in your web application.

### Infobip RTC Client

After you have received a token via HTTP API,
you are ready to instantiate the [`InfobipRTC`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC) client.
You can do that using these commands:

```javascript
let token = obtainToken(); // here you call '/webrtc/1/token'
let options = {debug: true}
let infobipRTC = new InfobipRTC(token, options);
```

Note that this doesn't actually connect to Infobip platform,
it just creates a new instance of [`InfobipRTC`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC).
Connection is made via the [`connect`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#connect) method.
Before connecting, it is useful to set up event handlers, so you can perform something when the connection is set up,
when the connection is lost, etc.
Events are set up via [`on`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#on) method:

```javascript
infobipRTC.on('connected', function (event) {
    console.log('Connected with identity: ' + event.identity);
});
infobipRTC.on('disconnected', function (event) {
    console.log('Disconnected!');
});
```

Now you are ready to connect:

```javascript
infobipRTC.connect();
```

### Making a call

You can call another subscriber if you know their identity.
It is done via the [`call`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call) method:

```javascript
let outgoingCall = infobipRTC.call('Alice');
```

Or if you want to initiate video call:

```javascript
let outgoingCall = infobipRTC.call('Alice', CallOptions.builder().setVideo(true).build());
```

As you can see, the [`call`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call) method
returns an instance of [`OutgoingCall`](https://github.com/infobip/infobip-rtc-js/wiki/OutgoingCall) as the result.
With it, you can track the status of your call and respond to events. Similar to the client,
you can set up event handlers, so you can do something when the called subscriber answers the call,
rejects it, the call is ended, etc. You set up event handlers with the following code:

```javascript
outgoingCall.on('ringing', function (event) {
    console.log('Call is ringing on Alice\'s device!');
});
outgoingCall.on('established', function (event) {
    console.log('Alice answered call!');
});
outgoingCall.on('hangup', function (event) {
    console.log('Call is done! Status: ' + JSON.stringify(event.status));
});
outgoingCall.on('error', function (event) {
    console.log('Oops, something went very wrong! Message: ' + JSON.stringify(event));
});
```

The most important part of the call is definitely the media that travels between the subscribers.
It can be handled in an `established` event where you have the remote media which can be streamed into your HTML page.
This is an example of how you can use it:

```javascript
<audio id="remoteAudio" autoplay/>

outgoingCall.on('established', function (event) {
    console.log('Alice answered call!');
    document.getElementById('remoteAudio').srcObject = event.remoteStream;
});
```

In case of video call, you should set to your video HTML elements both the stream from your camera,
and the stream that you got. Example of how you can use it:

```javascript
<video id="localVideo" autoplay muted/>
<video id="remoteVideo" autoplay/>

outgoingCall.on('established', function (event) {
    console.log('Alice answered video call!');
    document.getElementById('localVideo').srcObject = event.localStream;
    document.getElementById('remoteVideo').srcObject = event.remoteStream;
});
```

When event handlers are set up and the call is established, there are a few things that you can do with the actual call.
One of them, of course, is to hang up.
That can be done via the [`hangup`](https://github.com/infobip/infobip-rtc-js/wiki/Call#hangup) method on the call,
and after that, both parties will receive the `hangup` event upon hang up completion.

```javascript
outgoingCall.hangup();
```

You can simulate digit press during the call by sending DTMF codes (Dual-Tone Multi-Frequency).
This is achieved via [`sendDTMF`](https://github.com/infobip/infobip-rtc-js/wiki/Call#sendDTMF) method.
Valid DTMF codes are digits from `0-9`, `*` and `#`.

```javascript
outgoingCall.sendDTMF('*');
```

During the call, you can also mute (and unmute) your audio:

```javascript
outgoingCall.mute(true);
```

To check if the audio is muted, call the
[`muted`](https://github.com/infobip/infobip-rtc-js/wiki/Call#muted) method in the following way:

```javascript
let audioMuted = outgoingCall.muted();
```

Also, you can check the [`call status`](https://github.com/infobip/infobip-rtc-js/wiki/CallStatus):

```javascript
let status = outgoingCall.status();
```

Also, you can check information such as [`duration`](https://github.com/infobip/infobip-rtc-js/wiki/Call#duration),
[`start time`](https://github.com/infobip/infobip-rtc-js/wiki/Call#startTime),
[`establish time`](https://github.com/infobip/infobip-rtc-js/wiki/Call#establishTime) and
[`end time`](https://github.com/infobip/infobip-rtc-js/wiki/Call#endTime) by calling these methods:

```javascript
let duration = outgoingCall.duration();
let startTime = outgoingCall.startTime();
let establishTime = outgoingCall.establishTime();
let endTime = outgoingCall.endTime();
```

### Receiving a call

Besides making outgoing calls, you can also receive incoming calls.
In order to do that, you need to register the `incoming-call` event handler of
[`InfobipRTC`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC) client.
That is where you define the behavior of the incoming call.
One of the most common things to do is to show Answer and Reject options on the UI.
For the purpose of this guide, let's look at an example that answers the incoming call as soon as it arrives:

```javascript
infobipRTC.on('incoming-call', function (incomingCallEvent) {
    const incomingCall = incomingCallEvent.incomingCall;
    console.log('Received incoming call from: ' + incomingCall.source().identity);

    incomingCall.on('established', function () {
    });
    incomingCall.on('hangup', function () {
    });

    incomingCall.accept(); // or incomingCall.decline();
});
```

### Calling phone number

It is similar to calling the regular WebRTC user, you just use the
[`callPhoneNumber`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#callPhoneNumber) method instead of
[`call`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call).
This method accepts an optional second parameter, where you define the `from` parameter.
Its value will be displayed on the calling phone device as the Caller ID.
The result of the [`callPhoneNumber`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#callPhoneNumber) is the
[`OutgoingCall`](https://github.com/infobip/infobip-rtc-js/wiki/OutgoingCall) with which you can do everything
as when using the [`call`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call) method:

* Example of calling phone number with `from` defined:

```javascript
let outgoingCall = infobipRTC.callPhoneNumber('41793026727', CallPhoneNumberOptions.builder().setFrom('33712345678').build());
```

* Example of calling phone number without `from` defined:

```javascript
let outgoingCall = infobipRTC.callPhoneNumber('41793026727');
```

### Conference call

You can have a conference call with other participants that are also in the same conference room.
The conference call will start as soon as at least one participant joins.

Conference call is in the beta stage and available for both video and audio, with a maximum limit of 12 participants.

Joining the room is done via the
[`joinConference`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#joinConference) method:

```javascript
let conference = infobipRTC.joinConference('conference-demo');
```

Or if you want to join the conference with your video:

```javascript
let conference = infobipRTC.joinConference('conference-demo', ConferenceOptions.builder().setVideo(true).build());
```

By default, if you lose an internet connection during the conference call, it will be terminated, and the rest of users
will receive a `user-left` event.
But if you start the conference with the `autoReconnect` flag set to `true`, after losing the connection, you will
receive a `reconnecting` event
and we will immediately try to connect you back. Once we succeed, you will get a `reconnected` event.

```javascript
let conference = infobipRTC.joinConference('conference-demo', ConferenceOptions.builder().setAutoReconnect(true).build());
```

As you can see, that method returns an instance of
[`Conference`](https://github.com/infobip/infobip-rtc-js/wiki/Conference) as the result.
With it, you can track the status of your conference call,
do some actions (mute, share the screen, start sending video...) and respond to events.

After the user successfully joined the conference, the `joined` event will be emitted.
It contains an audio media stream and a list of users that are already at the conference.
You should implement an event handler for it, where the stream should be set to the audio HTML element,
and the other conference participants might be shown to the user.

Here is an example of how to
handle [`conference events`](https://github.com/infobip/infobip-rtc-js/wiki/Conference#on-conference).

Let's assume that we do have an audio HTML element with the id `conferenceAudio` and video HTML elements with the ids
`localVideo` and `localScreenShare`.

```javascript
conference.on('joined', function (event) {
    $('#conferenceAudio').srcObject = event.stream;
    var users = event.users.map(user => user.identity).join(", ")
    console.log('You have joined the conference with ' + event.users.length + " more participants: " + users);
});
conference.on('left', function (event) {
    console.log('You have left the conference.');
});
conference.on('error', function (event) {
    console.log('Error!');
});
conference.on('reconnecting', function (event) {
    console.log('You left the conference due to connectivity issues. We will try to reconnect you!');
});
conference.on('reconnected', function (event) {
    $('#conferenceAudio').srcObject = event.stream;
    var users = event.users.map(user => user.identity).join(", ")
    console.log('You have beed reconnected to the conference with ' + event.users.length + " more participants: " + users);
});
conference.on('user-joined', function (event) {
    console.log(event.user.identity + ' user joined.');
});
conference.on('user-left', function (event) {
    console.log(event.user.identity + ' user left.');
});
conference.on('user-muted', function (event) {
    console.log(event.user.identity + ' user muted himself.');
});
conference.on('user-unmuted', function (event) {
    console.log(event.user.identity + ' user unmuted himself.');
});
conference.on('user-talking', function (event) {
    console.log(event.user.identity + ' user started talking.');
});
conference.on('user-stopped-talking', function (event) {
    console.log(event.user.identity + ' user stopped talking.');
});
conference.on('local-camera-video-added', function (event) {
    $('#localVideo').srcObject = event.stream;
});
conference.on('local-camera-video-removed', function (event) {
    $('#localVideo').srcObject = null;
});
conference.on('local-screenshare-added', function (event) {
    $('#localScreenShare').srcObject = event.stream;
});
conference.on('local-screenshare-removed', function (event) {
    $('#localScreenShare').srcObject = null;
});
```

The next two events are fired when another user adds or removes the video.
You should implement these event handlers in order to add and/or remove an HTML video element with a media stream.

```javascript
conference.on('user-camera-video-added', function (event) {
    // add a new HTML video element with id remoteVideo-event.user.identity
    $('#remoteVideo-' + event.user.identity).srcObject = event.stream;
});
conference.on('user-camera-video-removed', function (event) {
    // remove the HTML video element with id remoteVideo-event.user.identity
});
```

The next two events are fired when another user starts or stops sharing screen.
You should implement these event handlers in order to add and/or remove an HTML video element with a media stream.

```javascript
conference.on('user-screenshare-added', function (event) {
    // add a new HTML video element with id remoteScreenShare-event.user.identity
    $('#remoteScreenShare-' + event.user.identity).srcObject = event.stream;
});
conference.on('user-screenshare-removed', function (event) {
    // remove the HTML video element with id remoteScreenShare-event.user.identity
});
```

When event handlers are set up and the conference call is established,
there are a few things that you can do with the actual conference.

One of them, of course, is to leave it. That can be done via the
[`leave`](https://github.com/infobip/infobip-rtc-js/wiki/Conference#leave) method at the conference.
Other participants will receive the `user-left` event upon leave completion.

```javascript
conference.leave();
```

During the conference call, you can also mute (and unmute) your audio, by calling the
[`mute`](https://github.com/infobip/infobip-rtc-js/wiki/Conference#mute) method in the following way:

```javascript
conference.mute(true);
```

During the conference call, you can start or stop sending your video, by calling the
[`cameraVideo`](https://github.com/infobip/infobip-rtc-js/wiki/Conference#cameraVideo) method in the following way:

```javascript
conference.cameraVideo(true | false);
```

After this method, the `local-camera-video-added` or `local-camera-video-removed` event is fired.

During the conference call, you can start or stop sharing your screen, by calling the
[`screenShare`](https://github.com/infobip/infobip-rtc-js/wiki/Conference#screenShare) method in the following way:

```javascript
conference.screenShare(true | false);
```

After this method, the `local-screenshare-added` or `local-screenshare-removed` event is fired.

### Media Device information

Beside getting the information about one's media devices anytime after the user is registered to Infobip's platfom,
or during the call, you can get these infos using the static methods
in [`RTCMediaDevice`](https://github.com/infobip/infobip-rtc-js/wiki/RTCMediaDevice).

```javascript
let RTCMediaDevice = require('infobip-rtc');
```

For audio input devices you can use the static
[`RTCMediaDevice.getAudioInputDevices`](https://github.com/infobip/infobip-rtc-js/wiki/RTCMediaDevice#getAudioInputDevices)
method:

```javascript
RTCMediaDevice.getAudioInputDevices().then(
    mediaDevices => {
        mediaDevices.forEach(device => {
            console.log(device.label);
        });
    }
);
```

For audio output devices you can use the static
[`RTCMediaDevice.getAudioOutputDevices`](https://github.com/infobip/infobip-rtc-js/wiki/RTCMediaDevice#getAudioOutputDevices)
method:

```javascript
RTCMediaDevice.getAudioOutputDevices().then(
    mediaDevices => {
        mediaDevices.forEach(device => {
            console.log(device.label);
        });
    }
);
```

For video input devices you can use the static
[`RTCMediaDevice.getVideoInputDevices`](https://github.com/infobip/infobip-rtc-js/wiki/RTCMediaDevice#getVideoInputDevices)
method:

```javascript
RTCMediaDevice.getVideoInputDevices().then(
    mediaDevices => {
        mediaDevices.forEach(device => {
            console.log(device.label);
        });
    }
);
```

Also, you can get the stream of a specific device using the static
[`RTCMediaDevice.getMediaStream`](https://github.com/infobip/infobip-rtc-js/wiki/RTCMediaDevice#getMediaStream) method:

```javascript
let mediaDevices = await RTCMediaDevice.getAudioInputDevices();
let device = mediaDevices[0];
let stream = await RTCMediaDevice.getMediaStream(device.deviceId);
$('#audio').srcObject = stream;
//...
RTCMediaDevice.closeMediaStream(stream);
```

### Browser Compatibility

We support up to 5 most recent versions of these browsers (unless otherwise indicated):

|                         |                               | ![chrome](assets/icons/chrome.png)  | ![firefox](assets/icons/firefox.png) | ![safari](assets/icons/safari.png)  | ![edge](assets/icons/edge.png)      | ![opera](assets/icons/opera.png)  | ![opera](assets/icons/explorer.png) |
|------------------------:|------------------------------:|:----------------------------:|:-----------------------------:|:----------------------------:|:----------------------------:|:--------------------------:|:----------------------------:|
|                         |                               | **Google Chrome**            | **Mozilla Firefox**           | **Safari***                  | **Microsoft Edge****           | **Opera**                  | **Internet Explorer**        |
| **Android**             | ![android](assets/icons/android.png) | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)      | ![chrome](assets/icons/no.png)      | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)   | ![chrome](assets/icons/no.png)      |
| **iOS*****                | ![ios](assets/icons/ios.png)         | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)      | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)   | ![chrome](assets/icons/no.png)      |
| **Linux**               | ![linux](assets/icons/linux.png)     | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)      | ![chrome](assets/icons/no.png)      | ![chrome](assets/icons/no.png)      | ![chrome](assets/icons/yes.png)   | ![chrome](assets/icons/no.png)      |
| **macOS**               | ![macOs](assets/icons/mac.png)       | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)      | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)   | ![chrome](assets/icons/no.png)      |
| **Windows**             | ![windows](assets/icons/windows.png) | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)      | ![chrome](assets/icons/no.png)      | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)   | ![chrome](assets/icons/no.png)      |

\* WebRTC support in Safari started with version 11.

\** WebRTC support in Microsoft Edge for Android, iOS and macOS started with Chromium-based version 79.
InfobipRTC is supported only for Chromium-based Microsoft Edge versions for Windows also.

\*** WebRTC support in browsers other than Safari started with iOS version 14.3.

> **Note**: Mobile browsers are not able to receive calls and maintain call connectivity in the background.
> We recommend using [iOS](https://github.com/infobip/infobip-rtc-ios)
> and [Android](https://github.com/infobip/infobip-rtc-android) SDKs
> for creating mobile WebRTC Applications.
