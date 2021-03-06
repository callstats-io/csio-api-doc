# API Enums

## Enumeration of fabricUsage

Usage       | Description
------------- | -----------
`multiplex`  | Describes a _PeerConnection_ carrying multiple media streams on the same port.
`audio`  | Describes an audio-only _PeerConnection_.
`video`  | Describes a video-only _PeerConnection_.
`screen`  | Describes a screen-sharing _PeerConnection_.
`data`  | Describes a _PeerConnection_ with only DataChannels.
`unbundled`  | Describes a _PeerConnection_ carrying media streams on different ports.

When using a single _PeerConnection_ between a pair of userIDs for sending and receiving audio and video, application MUST use `multiplex`.

<!-- Currently monitoring DATA traffic is NOT SUPPORTED, because the browser does not yet implement any DataChannel statistics. -->


## Enumeration of fabricEvent

Name  | Description
---------  | -----------
`fabricHold` | The fabric is currently not sending and receiving any media, but the connection is still active.
`fabricResume`  | The fabric is resuming communication with the remote endpoint.
`audioMute` | The fabric is currently not sending any Audio, but MAY be sending video.
`audioUnmute` | The fabric is resuming Audio communication.
`videoPause` | The fabric is currently not sending any Video, but MAY be sending audio.
`videoResume` | The fabric is resuming Video communication.
`fabricTerminated`  | The _PeerConnection_ is destroyed and is no longer sending or receiving any media.
`screenShareStart`  | The _PeerConnection_ started the screen sharing.
`screenShareStop`  | The _PeerConnection_ stopped the screen sharing.
`dominantSpeaker`  | The userID reports that it is the dominant speaker and not the remote participants.
`activeDeviceList` | The userID reports the active devices used by him during the conference.


## Enumeration of webRTCFunctions

Function Name  | Description
---------  | -----------
`getUserMedia`  | The failure occurred in getUserMedia function (added in callstats.js version 3.4.x).
`createOffer`  | The failure occurred in createOffer function.
`createAnswer`  | The failure occurred in createAnswer function.
`setLocalDescription`  | The failure occurred in setLocalDescription function.
`setRemoteDescription`  | The failure occurred in setRemoteDescription function.
`addIceCandidate`  | The failure occurred in addIceCandidate function.
`iceConnectionFailure`  | Ice connection failure detected by the application.
`signalingError`  | Signaling related errors in the application.
`applicationLog`  | Application related logs, this will not be considered as a failure.

## Enumeration of endpointType

Name  | Description
---------  | -----------
`peer` | The endpoint is a WebRTC client/peer.
`server`| The endpoint is a media server or a middle-box.

## Enumeration of transmissionDirection

Name  | Description
---------  | -----------
`sendonly` | PeerConnection is for sending only.
`receiveonly`| PeerConnection is for receiving only.
`sendrecv` | PeerConnection is for sending and receiving.
`inactive`| PeerConnection is inactive.

## Enumeration of userIDType

Name  | Description
---------  | -----------
`local` | Set the localUserID
`remote`| Set the remoteUserID

## csStatus Types

Name  | Description
---------  | -----------
`httpError`  | HTTP error.
`authError`  | Authentication failed, AppID or AppSecret is incorrect.
`success`  | The back-end has accepted the request and the endpoint is authenticated, or capable of sending measurement data.
`tokenGenerationError` | Application could not generate the JWT.

## Enumeration of callStatsAPIReturnStatus

Name      | Description
------------- | -----------
`success`  | The API call was successful.
`failure`  | The API call failed.
