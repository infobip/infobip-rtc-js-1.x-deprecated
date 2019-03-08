* [`hangup()`](#hangup)
* [`on(event: string, eventHandler: any)`](#on)
* [`mute(shouldMute: boolean)`](#mute)
* [`sendDTMF(tone: string)`](#sendDTMF)

<a name="hangup"></a>
## `hangup()`

### Description
Hangs up call, which ends up in both parties receiving `hangup` event, after hangup is processed by Infobip WebRTC platform.

### Arguments
none

### Returns
none

### Example
```
let infobipRTC = new InfobipRTC('2e29c3a0-730a-4526-93ce-cda44556dab5', {url: 'portunus.infobip.com'});
let call = infobipRTC.call('Alice');
call.on('established', function() {
 call.hangup();
});
```

<a name="on"></a>
## `on(eventName, eventHandler)`

### Description
Configures event handler for call events. There are 2 event types in Infobip RTC API, ones that are configured globally, per connection basis (RTC events), and ones that are configured per call basis (call events). This one handles call events.

### Arguments
* `eventName`: *string* - Name of call event. Allowed values are: `established`, `hangup`, `error`, `ringing`. `ringing` is for `OutgoingCall` only.
* `eventHandler`: *any* - Function that will be called when event occurs. Exactly one parameter is passed to function, when event occurs. For every event, there is set of fields that parameter will contain.  
** `established` - Contains localStream and remoteStream fields (media streams of both parties of the call).  
** `hangup` - Contains status field (instance of HangupStatus class, describing reason why call was hang up).  
** `error` - One of the ErrorCode, enum that helps you identify which error occurred.  
** `ringing` - Contains no fields.  

### Returns
none

### Example
```
let infobipRTC = new InfobipRTC('2e29c3a0-730a-4526-93ce-cda44556dab5', {url: 'portunus.infobip.com'});
let call = infobipRTC.call('Alice');
call.on('established', function() {
 console.log('Established!')
});
```

<a name="mute"></a>
## `mute(shouldMute)`

### Description
Toggles mute option.

### Arguments
* `shouldMute`: *boolean* - Whether call should be muted after action or not.

### Returns
none

### Example
```
let infobipRTC = new InfobipRTC('2e29c3a0-730a-4526-93ce-cda44556dab5', {url: 'portunus.infobip.com'});
let call = infobipRTC.call('Alice');
call.on('established', function() {
call.mute(true);
});
```


<a name="sendDTMF"></a>
## `sendDTMF(tone)`

### Description
Simulates key-press by sending DTMF (Dual-Tone Multi-Frequency) entry.

### Arguments
* `tone`: *string* - One of the allowed DTMF characters (digits from `0` to `9`, `*` and `#`).

### Returns
none

### Example
```
let infobipRTC = new InfobipRTC('2e29c3a0-730a-4526-93ce-cda44556dab5', {url: 'portunus.infobip.com'});
let call = infobipRTC.call('Alice');
call.on('established', function() {
 call.sendDTMF('1');
});
```