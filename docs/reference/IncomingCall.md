# `extends` [`Call`](./Call)

* [`accept()`](#accept)
* [`decline()`](#decline)

<a name="accept"></a>
## `accept()`

### Description
Accepts incoming call, which ends up in both parties receiving `established` event, after accept is processed by Infobip WebRTC platform.

### Arguments
none

### Returns
none

### Example
```
let infobipRTC = new InfobipRTC('2e29c3a0-730a-4526-93ce-cda44556dab5', {url: 'portunus.infobip.com'});
infobipRTC.on('incoming-call', function(call) {
 call.accept();
});
```

<a name="decline"></a>
## `decline()`

### Description
Declines incoming call, which ends up in both parties receiving `hangup` event with proper status, after decline is processed by Infobip WebRTC platform.

### Arguments
none

### Returns
none

### Example
```
let infobipRTC = new InfobipRTC('2e29c3a0-730a-4526-93ce-cda44556dab5', {url: 'portunus.infobip.com'});
infobipRTC.on('incoming-call', function(call) {
 call.decline();
});
```