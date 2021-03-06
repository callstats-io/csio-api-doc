# Integrating with your App


## Step 1: Include callstats.js

<aside class="error">
You can track your integration progress from our <a href= "https://dashboard.callstats.io/apps/integration"> dashboard </a>.
</aside>


```javascript
  <script src="https://api.callstats.io/static/callstats.min.js"></script>
```
> Everything in the `callstats.js` is scoped under the callstats namespace, hence create the object.

```javascript
  var callstats = new callstats();
```
Add the `callstats.js` in the HEAD tag.

<aside class="error">
If you are using require.js, please refer to the following <a href="#loading-with-require-js"> section </a>
</aside>

## Step 2: Initialize() with AppSecret

```javascript
  //initialize the app with application tokens
  var AppID     = "YOUR APPLICATION ID";
  var AppSecret = "YOUR APPLICATION SECRET";

  //localUserID is generated or given by the origin server
  callstats.initialize(AppID, AppSecret, localUserID, csInitCallback, csStatsCallback, configParams);
```

After the user is authenticated with the origin server (or when the page loads), call `initialize()` with appropriate parameters (see [API section](#api)).  Check the callback for errors.  If the authentication succeeds, `callstats.js` will receive a valid authentication token to make subsequent API calls.

For more information on callbacks, please refer to [csInitCallback](#csinitcallback) and [csStatsCallback](#csstatscallback). Also have a look at [step 8](#step-8-optional-handling-stats-from-statscallback) for csStatsCallback data handling.

ALTERNATIVE: If you are interested in using the third-party authentication, see the details described in a [later section](#third-party-authentication).

## Step 3: addNewFabric()

```javascript
  //adding Fabrics
  var pc_config = {"iceServers": [{url: "stun:stun.example.org:3478"}]};
  var pcObject = new RTCPeerConnection(pc_config);

  function pcCallback (err, msg) {
    console.log("Monitoring status: "+ err + " msg: " + msg);
  };

  // pcObject is created, tell callstats about it
  // pick a fabricUsage enumeration, if pc is sending both media and data: use multiplex.

  var usage = callstats.fabricUsage.multiplex;
  var fabricAttributes = {
    remoteEndpointType:   callstats.endpointType.peer,
    fabricTransmissionDirection:  callstats.transmissionDirection.sendrecv
    };

  //remoteUserID is the recipient's userID
  //conferenceID is generated or provided by the origin server (webrtc service)
  callstats.addNewFabric(pcObject, remoteUserID, usage, conferenceID, fabricAttributes, pcCallback);

```

When creating a _PeerConnection_, call `addNewFabric()` with appropriate parameters (see [API section](#callstats-addnewfabric)). It is important to make the request only after the _PeerConnection_ is created. The _PeerConnection_ object MUST NOT be "undefined" or NULL because `callstats.js` uses [`getStats()`](http://dev.w3.org/2011/webrtc/editor/webrtc.html#statistics-model) to query the metrics from the browser internals. The application SHOULD call `addNewFabric()` immediately after the _PeerConnection_ object is created.

Time stamp of `addNewFabric()` is used as a reference point to calculate fabric failure delay or fabric setup delay:

<aside class="error">
<ul>

<li> Fabric failure delay = timestamp of fabricSetupFailed - timestamp of addNewFabric</li>
<li> Fabric setup delay = timestamp of fabricSetup - timestamp of addNewFabric </li>

</ul>
</aside>

In any WebRTC endpoint, where multiple _PeerConnections_ are created between each participant (e.g., audio and video sent over different _PeerConnections_ or a mesh call), the `addNewFabric()` MUST be called for each _PeerConnection_.


## Step 4: reportError()

```javascript
  //adding Fabrics
  var pc_config = {"iceServers": [{url: "stun:stun.example.org:3478"}]};
  var pcObject = new RTCPeerConnection(pc_config);

  function pcCallback (err, msg) {
    console.log("Monitoring status: "+ err + " msg: " + msg);
  };

  function createOfferError(err) {
    callstats.reportError(pcObject, conferenceID, callstats.webRTCFunctions.createOffer, err);
  }

  // remoteUserID is the recipient's userID
  // conferenceID is generated or provided by the origin server (webrtc service)
  // pcObject is created, tell callstats about it
  // pick a fabricUsage enumeration, if pc is sending both media and data: use multiplex.

  var usage = callstats.fabricUsage.multiplex;
  callstats.addNewFabric(pcObject, remoteUserID, usage, conferenceID, pcCallback);

  // let the "negotiationneeded" event trigger offer generation
  pcObject.onnegotiationneeded = function () {
    // create offer
    pcObject.createOffer().then(
      localDescriptionCreatedCallback,
      createOfferErrorCallback
    );
  }
```

Sometimes WebRTC endpoints fail to establish connectivity, this may occur when user-agents and/or bridges implement differing flavors of the Session Description Protocol (SDP) or may not support some features that others implement.

The WebRTC APIs either have a callback or a Promise associated to them. Since `callstats.js ver. 3.2.x`, WebRTC applications can use `reportError()` to capture at which stage the negotiation fails and pass on the [DomError](http://www.w3.org/TR/dom/#interface-domerror) returned by the callback or Promise to callstats.io. The failure reason will appear both in the conference time-line and aggregate on the main dashboard. See Section enumerating [WebRTC functions](#enumeration-of-wrtcfuncnames) for details. The example below reports error when creating an SDP offer:


<aside class="success">
Congratulations! You have now completed the basic integration steps, read more for advanced features!
</aside>

## Step 5: (OPTIONAL) sendFabricEvent()

```javascript
// send fabricEvent: videoPause
callstats.sendFabricEvent(pcObject, callstats.fabricEvent.videoPause, conferenceID);

// devices are returned by the
/*
var devices = navigator.mediaDevices.getUserMedia({
  audio: true,
  video: true
});
//a typical device looks like:
//device= {  "deviceId":"default","kind":"videoinput","label":"FaceTime HD Camera","groupId":"2004946474"}
*/

var eventData = {
  deviceList: devices // array of active device
};

// send fabricEvent: activeDeviceList
callstats.sendFabricEvent(pcObject, callstats.fabricEvent.activeDeviceList, conferenceID, eventData);
```

During the conference, users might perform several actions impacting the measurements and conference analysis. The user might mute the audio or switch off the camera or do screen sharing during a conference. These events can directly impact the measurement data (For example, you can see a significant drop in throughput when camera is switched off). For the list of all possible conference events, please refer [here](#enumeration-of-fabricevent)

Send the appropriate `fabricEvent` via `sendFabricEvent()`.

<!-- - send `fabricSetup` after [`onaddstream` event is fired by the WebRTC API](http://dev.w3.org/2011/webrtc/editor/webrtc.html)
  and the endpoint starts to render media. The callstats javascript expects both local
  and remote endpoints to generate this event. After this event is fired, the
  callstats javascript begins performance monitoring and sending data to the
  [callstats.io]({{site.callstats.backend-url}}) backend. -->

- `fabricSetup` and `fabricSetupFailed` has been **deprecated** in v2.1.0 and v3.10.0,
  respectively, these events are now generated automatically by the JS library.

<!-- - send `fabricFailed` when a call fails to connect to the remote peer or
  to the conferencing server. For example, failing to traverse a NAT or
  middlebox (ICE failure). This event MUST be reported by a local endpoint
  when the call fails without engaging the remote endpoint. In cases where
  the call fails after signaling to the remote endpoint succeeds, each
  endpoint MUST report the event independently. -->

- send `fabricTerminated` when an endpoint or participant disconnects from
  the conference, it notifies callstats.js to stop monitoring
  the local _PeerConnection_. Depending on the implementation of the
  hangup in your WebRTC application (may have to rely on signaling), the
  remote endpoint sends a `fabricTerminated` event before destroying its
  local _PeerConnection_ object.
  callstats.io monitors each
  _PeerConnection_ in real-time, and generates summary statistics when the
  participant leaves. The summary of statistics for each conference is aggregated
  when all the participant have left. If no `fabricTerminated` event is received,
  callstats.io will summarize
  and aggregate the summary statistics _30 seconds_ after the last measurement
  for a conference is received.

- send `fabricHold` or `fabricResume` whenever the user holds and resumes the call.
  This is usually done when a user gets multiple incoming conference calls, and has
  to stop transmitting (hold) on one conference call to transmit on the other, and
  then returns to earlier call to resume transmitting (unhold).

- send `dominantSpeaker` when a particular userID appears to be the only participant
  speaking. Typically, each endpoint calculates the dominant speaker over the set of
  participants in a sliding time-window (say, 10 seconds). Then the endpoint that
  notices that it is the dominant speaker sends the event.

- send `activeDeviceList` whenever the audio input, audio output, or video input
  device changes.

## Step 6: (OPTIONAL) associateMstWithUserID()

```javascript
  // After O/A is complete, i.e., onAddStream is fired
  var localUserID  = "Alice";
  var remoteUserID = "Bob";
  var conferenceID = "AliceAndBobAndCharlie";
  var mstLabel = "front-camera";
  // SSRC1 is the SSRCs of the local video stream
  // SSRC2 is the SSRC of the remote video stream, usually received in the remote SDP
  // mstLabel is a developer provided string that lets them identify
  // various tracks (e.g., front-camera, back-camera, without looking at the
  // configurations of the individual MSTs). In this example, we assume it is the
  // front-camera.
  callstats.associateMstWithUserID(pc, localUserID, conferenceID, ssrc1, mstLabel);
  callstats.associateMstWithUserID(pc, remoteUserID, conferenceID, ssrc2, mstLabel);
```

When interacting with the conference server, the developer is most likely going to use the name or identifier associated with the conference server as the `remoteUserID`. A typical conference bridge (for example, [Jitsi Videobridge](https://jitsi.org/Projects/JitsiVideobridge)) transmits [multiple media stream tracks within a peer connection](https://hacks.mozilla.org/2015/06/firefox-multistream-and-renegotiation-for-jitsi-videobridge/). In which case, using a remote participant’s userID is impractical as there maybe several participants.

Since `callstats.js ver. 3.3.x`, we allow mapping [Synchronization Source Identifier (SSRC)](http://tools.ietf.org/html/rfc3550#section-5.1) for a mediastreamtrack to a userID (both local and remote). By default the local and remote _MediaStreamTracks_ are automatically mapped to the localUserID and remoteUserID. With `associateMstWithUserID()`, you can override the actual local and remote userIDs to the correct association. If the DOM identifiers of the video tags associated to each participant, callstats.js will calculate better quality scores for each participant. The code example shows how the API can be integrated:

<img src="/images/2015-mst-association.gif" alt="Associationg userID with MST" width="700"/>


More discussion related to the motivation of `associateMstWithUserID()` is covered in the following [blog post](/2015/07/17/api-update-handling-multiple-media-stream-tracks-callstats/).


## Step 7: (OPTIONAL) sendUserFeedback()

```javascript
  var overallRating = 4; // 1-5 rating, typically according to MOS scale.
  var feedback = {
    "userID": localUserID, //mandatory
    "overall": overallRating, //mandatory
  };
  callstats.sendUserFeedback(conferenceID, feedback, pcCallback);
```

The developers are expected to design an appropriate UI to get user input on quality at the end of the call. Typically, services collect user feedback based on the Mean Opinion Score (MOS). However, it is not neccessary to use all values of the  MOS scale, for example a service using only 2 point scale: it can associate 1 and 5 to bad and excellent, respectively and not use the values 2 to 4.


## Step 8: (OPTIONAL) Handling stats from csStatsCallback()

The developers can handle the stats received from csStatsCallback function in a way suitable to their application. It can be used for displaying bitrate or based on the conference quality indicators applications can change their settings etc. For more details check this [blog post](/2015/08/24/statscallback-webrtc-media-quality-status/).

## Step 9: (OPTIONAL) Submitting application logs

The developers can send application error logs using `reportError()` API and track them on callstats.io dashboard. The logs will help in debugging the corresponding conferences. The `error` can be an object or a string.

```javascript
var error1 = {
    message: "Error message",
    error: "Error 1",
    stack: "stack trace for the error"
};

callstats.reportError(pc, confID, callstats.webRTCFunctions.applicationLog, error1);
```

```javascript
error2 = "application error ";

callstats.reportError(pc, confID, callstats.webRTCFunctions.applicationLog, error2);
```
<aside class="error">
Please note that the application log size is limited to 20KB. Any application log greater than 20KB will be truncated to 20KB and a warning message will be displayed on the console log.
</aside>

## Step 10: (OPTIONAL) Obtaining the default configuration

```javascript
// example: default configuration callback

function csDefaultConfigCallback(config) {
    var pc = new RTCPeerConnection(config.peerConnection);
    getUserMedia(config.media);
}
// Setting the default configuration callback

callstats.on(“defaultConfig”, csDefaultConfigCallback);

```

You can obtain the default configuration you have set on the dashboard via default configuration callback. The default configuration can be set for the Peer Connection or the media. You can set the configuration by visiting the "App Settings" page and navigating to the "configuration" tab. The section [RTCPeerConnection configuration](https://www.w3.org/TR/webrtc/#rtcconfiguration-dictionary) and [Media constraints](https://www.w3.org/TR/mediacapture-streams/#mediastreamconstraints) can be used for entering the default configuration in JSON format.
