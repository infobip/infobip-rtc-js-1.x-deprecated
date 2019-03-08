# `extends` [`Call`](./Call)

* [`id`](#id)
* [`name`](#name)
* [`description`](#description)

<a name="id"></a>
## `id`: *int*
Status ID, every status has it's unique identifier.

## `name`: *string*
Status name. Name conforms programming languages's enum notation.

## `description`: *string*
Status description. Sentence that tells exactly what caused hangup.

## Possible Values
<table>
    <thead>
        <th>ID</th>
        <th>Name</th>
        <th>Description</th>
        <th>Note</th>
    </thead>
    <tbody>
        <tr>
            <td>0</td>
            <td>NO_ERROR</td>
            <td>No Error.</td>
            <td></td>
        </tr>
        <tr>
            <td>5002</td>
            <td>EC_VOICE_USER_BUSY</td>
            <td>The end user is currently busy and can not receive the Voice call.</td>
            <td></td>
        </tr>
        <tr>
            <td>5003</td>
            <td>EC_VOICE_NO_ANSWER</td>
            <td>The end user received a call but did not answer it.</td>
            <td></td>
        </tr>
        <tr>
            <td>5050</td>
            <td>WEBRTC_ERROR</td>
            <td>See note.</td>
            <td>Happens if something bad happened inside SDK during call that caused hangup. Description will contain error message or Unknown error. if message is not available.</td>
        </tr>
        <tr>
            <td>5403</td>
            <td>EC_VOICE_ERROR_FORBIDDEN</td>
            <td>Received request was rejected.</td>
            <td></td>
        </tr>
        <tr>
            <td>5404</td>
            <td>EC_VOICE_ERROR_DESTINATION_NOT_FOUND</td>
            <td>The server has definitive information that the user does not exist at the domain specified in the Request-URI.</td>
            <td></td>
        </tr>
        <tr>
            <td>5408</td>
            <td>EC_VOICE_ERROR_REQUEST_TIMEOUT</td>
            <td>There was no coverage for a specific destination number or the end user could not be found in time during the call.</td>
            <td></td>
        </tr>
        <tr>
            <td>5480</td>
            <td>EC_VOICE_ERROR_TEMPORARILY_NOT_AVAILABLE</td>
            <td>Destination address is currently not available.</td>
            <td></td>
        </tr>
        <tr>
            <td>5500</td>
            <td>EC_VOICE_INTERNAL_SERVER_ERROR</td>
            <td>The server could not fulfill the request due to some unexpected condition.</td>
            <td></td>
        </tr>
        <tr>
            <td>5503</td>
            <td>EC_VOICE_SERVICE_UNAVAILABLE</td>
            <td>The service failed to complete the request.</td>
            <td></td>
        </tr>
        <tr>
            <td>5XXX</td>
            <td>EC_UNKNOWN_ERROR</td>
            <td>See note.</td>
            <td>Happens if hangup is received with unrecognized SIP status. XXX from ID will be SIP status code, description will be SIP status line.</td>
        </tr>
    </tbody>
</table>