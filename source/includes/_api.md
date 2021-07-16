
#API


## callstats.initialize() with app secret

```javascript
var localUserID = {};
localUserID.userName = "Clark Kent";
localUserID.aliasName = "superman";
localUserID.loginID = "superman@dc"
```

```javascript
var additionalIDs = {
  customerID: "Customer Identifier. Example, walmart.",
  tenantID: "Tenant Identifier. Example, monster.",
  productName: "Product Name. Example, Jitsi.",
  meetingsName: "Meeting Name. Example, Jitsi loves callstats.",
  serverName: "Server/MiddleBox Name. Example, jvb-prod-us-east-mlkncws12.",
  pbxID: "PBX Identifier. Example, walmart.",
  pbxExtensionID: "PBX Extension Identifier. Example, 5625.",
  fqExtensionID: "Fully qualified Extension Identifier. Example, +71 (US) +5625.",
  sessionID: "Session Identifier. Example, session-12-34",
};
```

```javascript
var configParams = {
  disableBeforeUnloadHandler: true, // disables callstats.js's window.onbeforeunload parameter.
  applicationVersion: "app_version", // Application version specified by the developer.
  disablePrecalltest: true // disables the pre-call test, it is enabled by default.
  siteID: "siteID", // The name/ID of the site/campus from where the call/pre-call test is made.
  additionalIDs: additionalIDs, // additionalIDs object, contains application related IDs.
  collectLegacyStats: true //enables the collection of legacy stats in chrome browser
  collectIP: true //enables the collection localIP address
};
```

```javascript
callstats.initialize(AppID, AppSecret, localUserID, csInitCallback, csStatsCallback, configParams);
```

  Params  |  Argument | Type | Description
-----------  | ----------- | -------- | ----------
`AppID`  | Required | String | Application ID is obtained from callstats.io.
`AppSecret`  | Required | String | Application secret is obtained from callstats.io.
`localUserID`  | Required | String (128 bytes) or Object | it is provided by the developer and MUST NOT be null or empty.
`csInitCallback`  | Optional | Callback | asynchronously reports status of the protocol messages.
`csStatsCallback`  | Optional | Callback | asynchronously reports the conference statistics.
`configParams`  | Optional | JSON| it is the set of parameters to enable/disable certain features in the library.

### JSON for UserID (supported since v3.14)

In some cases, customers want to provide the actual username in addition to the alias to callstats.io. Since callstats.js version 3.14, it accepts userID both as a String or an object. Section on [generating userID](#generating-userid-conferenceid-and-jwt) provides more guidelines on choosing a `localUserID`.


  Keys  |  Required | Description
-----------  | -------- | ----------
`userName` | Yes | Strint of maximum lenth **128 characters**.
`aliasName` | Yes | String of maximum length **128 characters**.
`loginID` | No | String of maximum length **128 characters**.

### JSON for configParams

It provides developers a method to enable or disable certain features or functions within the `callstats.js` library. It is a javascript object with the following OPTIONAL key-value pairs. They are:

  Keys  |  Required | Description
-----------  | -------- | ----------
`disableBeforeUnloadHandler` | No | by default the value is `false`.
`applicationVersion` | No | String of maximum length **30 characters**.
`disablePrecalltest` | No | by default the value is `false`.
`siteID` | No | String (256 bytes).
`collectLegacyStats` | No | by default the value is `true`.
`additionalIDs` | No | JSON object.
`collectIP` | No | by default the value is `true`.

<aside class="error">
Setting `disableBeforeUnloadHandler` to `true` disengages callstats.js's `window.onbeforeunload` handler, and you will need to send the fabricTerminated event for each active PeerConnection. See more details on `fabricTerminated` [event](#step-5-optional-sendfabricevent)
</aside>

### JSON for additionalIDs

  Keys  |  Required | Description
-----------  | -------- | ----------
`customerID` | No | String (256 bytes) Example, walmart.
`tenantID` | No | String (256 bytes) Example, monster.
`productName` | No | String (256 bytes) Example, jitsi.
`meetingsName` | No | String (256 bytes) Example, jitsi loves callstats.
`serverName` | No | String (256 bytes) Example, jvb-prod-us-east-mlkncws12.
`pbxID` | No | String (256 bytes) Example, walmart.
`pbxExtensionID` | No | String (256 bytes) Example, 5625.
`fqExtensionID` | No | String (256 bytes) Example, +71 (US) +5625.
`sessionID` | No | String (256 bytes) Example, session-12-34.

## callstats.initialize() with third party authentication

- Authenticates with the callstats.io backend to setup a trusted relationship with it.

  Params  |  Argument | Type | Description
-----------  | ----------- | -------- | ----------
`AppID`  | Required | String | Application ID is obtained from callstats.io.
`tokenGenerator`  | Required | callback | Callback to generate token.
`localUserID`  | Required | String (128 bytes) or Object | it is provided by the developer and MUST NOT be null or empty.
`csInitCallback`  | Optional | callback | asynchronously reports status of the protocol messages.
`csStatsCallback`  | Optional | callback | asynchronously reports the conference statistics.  
`configParams`  | Optional | JSON| it is the set of parameters to enable/disable certain features in the library.


## callstats.addNewFabric()
```javascript
var fabricAttributes = {
  remoteEndpointType:   callstats.endpointType.peer,
  fabricTransmissionDirection:  callstats.transmissionDirection.sendrecv
};

 callstats.addNewFabric(pcObject, remoteUserID, fabricUsage, conferenceID, fabricAttributes, pcCallback);
```

- Indicates that the WebRTC application requests callstats.js to monitor the performance of the _PeerConnection_ between the two endpoints (represented by the corresponding UserIDs).

   Params  |  Argument | Type | Description
-----------  | ----------- | -------- | ----------
`pcObject`  | Required  |  Object | _PeerConnection_ object.
`remoteUserID`  |  Required  | String (128 bytes) or Object| It is generated by the origin server.
`fabricUsage`  |  Required  | Enum | Valid values are discussed in a later [section](#enumeration-of-fabricusage).
`conferenceID`   | Required  | String (256 bytes) | It is generated by the origin server.
`fabricAttributes`  | Optional  | Object |  Contains two attributes [section](#enumeration-of-endpointtype) and [section](#enumeration-of-transmissiondirection).
`pcCallback`  | Optional  | Callback | the callback asynchronously reports failure or success for pcObject.

<aside class="error">
<ul>
<li> <b>Please note!</b> To notify callstats.io of a fabric termination, the <a href="#callstats-sendfabricevent"> sendFabricEvent( ) </a> must be called with the value `fabricTerminated` in `fabricEvent`</li>
<li> fabricAttributes is optional and default value is peer and sendrecv</li>

</ul>
</aside>

## callstats.reportError()

```javascript
callstats.reportError(pcObject, conferenceID, callstats.webRTCFunctions.createOffer);
```

- Notifies the callstats.io back-end about conference setup failure reason.

   Params  |  Argument | Type | Description
-----------  | ----------- | -------- | ----------
`pcObject`  |  Required | Object | _PeerConnection_ object
`conferenceID`  |  Required | String (256 bytes)| It is generated by the origin server.
`webRTCFunctions`  |  Required | Enum | Name of the [WebRTC function](#enumeration-of-webrtcfunctions) that failed.
`domError`  |  Optional | Object | [DOMError object](https://developer.mozilla.org/en-US/docs/Web/API/DOMError)
`localSDP`  |  Optional  |Object| Local SDP collected when errors occur.
`remoteSDP`  |  Optional  | Object | Remote SDP collected when errors occur.

<aside class="error">
<ul>

<li> pcObject MAY be set to NULL when passing in errors that occur when getUserMedia() is called.</li>
<li> localSDP MAY be set to NULL, in case you do not want SDPs to be collected. </li>
<li> remoteSDP MAY be set to NULL, in case you do not want SDPs to be collected. </li>

</ul>
</aside>

## callstats.associateMstWithUserID()
```javascript
// Extracting SSRC from SDP
var validLine = RegExp.prototype.test.bind(/^([a-z])=(.*)/);
var reg = /^ssrc:(\d*) ([\w_]*):(.*)/;

pc.remoteDescription.sdp.split(/(\r\n|\r|\n)/).filter(validLine).forEach(function (l) {
        var type = l[0];
        var content = l.slice(2);
        if(type === 'a') {
          if (reg.test(content)) {
            var match = content.match(reg);
            if(($.inArray(match[1],ssrcs) === -1)) {
              ssrcs.push(match[1]);
            }
          }
        }
      });

    ssrcs.forEach(function(ssrc) {
      window.callstats.associateMstWithUserID(pcObject, userID, conferenceID, ssrc, usageLabel, associatedVideoTag);
    });
```

- Maps the SSRC to the userID that generated it. This is useful when multiple _MediaStreamTracks_ (MSTs) are sent or received in a single _PeerConnection_.

   Params  |  Argument | Type | Description
-----------  | ----------- | -------- | ----------
`pcObject`  | Required  | Object | _PeerConnection_ object.
`userID`  | Required  | String (128 bytes) or Object | It is generated by the origin server.
`conferenceID`  | Required | String (256 bytes)| It is generated by the origin server.
`SSRC`  | Required | Object |Synchronization Source Identifier, as defined in [RFC3550](https://tools.ietf.org/html/rfc3550).
`usageLabel`  | Required | String (20 bytes) |it is generated by the origin server.
`associatedVideoTag`  | Optional | String | handler to the user's video tag.

<aside class="error">
<ul>

<li> `userID` is `localUserID` for local MSTs and `remoteUserID` for remote MSTs.</li>
<li> SSRC is typically generated by the user-agent and MUST not be null or empty.</li>
<li> usageLabel can be set to front-camera, back-camera, screen, audio-in, mic, system-sound, enumeration-of-webrtcfunctions or anything that allows stream usage identification. </li>
<li> `associatedVideoTag` is the DOM element name, if you do not provide `associatedVideoTag` then callstats.js is not able to collect <b>Time-to-First-Media</b> metric</li>

</ul>
</aside>

## callstats.sendFabricEvent()
- Notifies the callstats.io back-end about user specific events (e.g., 'Hold', 'Mute', etc.) on a _PeerConnection_. It is usually generated due to local user interaction at a _PeerConnection_.

   Params  |  Argument | Type | Description
-----------  | ----------- | -------- | ----------
`pcObject`  | Required| Object |  _PeerConnection_ object.
`fabricEvent`  | Required | Enum| with valid values discussed in a later [section](#enumeration-of-fabricevent).
`conferenceID` | Required| String (256 bytes) | It is generated by the origin server.
`eventData` | Optional| Object | event related data.
<!---
## callstats.reportUserIDChange()
```javascript

callstats.reportUserIDChange(pcObject, conferenceID, newUserID, callstats.userIDType.local)

```
- This API can be used to change local or remote userID's during the conference.

   Params  |  Argument | Type | Description
-----------  | ----------- | -------- | ----------
`pcObject`  | Required  |  Object | _PeerConnection_ object.
`conferenceID`   | Required  | String (256 bytes) | It is generated by the origin server.
`newUserID`  |  Required  | String (128 bytes) or Object| It is generated by the origin server.
`userIDType`  | Required   | Enum | with valid values discussed in a later [section](#enumeration-of-useridtype).
--->

## callstats.sendUserFeedback()

- Send the feedback on conference performance indicated by the user.

   Params  |  Argument | Type | Description
-----------  | ----------- | -------- | ----------
`conferenceID`  | Required | String (256 bytes)| It is generated by the origin server.
`feedback`  | Required | Object| JSON object.
`pcCallback`  | Optional | Callback | the callback asynchronously reports failure or success of feedback submission.


### JSON for `feedback`

  Keys  |  Required | Type | Description
-----------  | -------- | -------- |----------
`userID` |  Required | String (128 bytes) | It is generated by the origin server. Discussed in a [later section](#generating-userid-conferenceid-and-jwt).
`overall` | Required | Integer (between 1-5) | Typically the scores correspond to the [mean opinion score](https://en.wikipedia.org/wiki/Mean_opinion_score), in this case this value represents the overall quality perceived by the userID.
`audio`, `video`, `screen` | Optional | Integer (between 1-5) | Similar to the definition of `overall`, except these values correspond to specific types of media streams.
`comment` | Optional | String | Detailed user feedback.

## callstats.on()

- The "on" function is used to set the callbacks for events.

   Params  |  Argument | Type | Description
-----------  | ----------- | -------- | ----------
`eventName`  | Required | String | The allowed values are "preCallTestResults", "defaultConfig", "recommendedConfig", "connectionRecommendation", "error", "mediaDisruption" and "stats".
`csEventCallback`  | Required | Callback | The callback asynchronously provides new event data whenever it is available.

```javascript
callstats.on(eventName, csEventCallback);

```

## callstats.attachWifiStatsHandler()

- The "attachWifiStatsHandler" function is used to set the callbacks for WiFi stats. You can send WiFi stats to callstat.io and visualize the stats on the dashboard.

   Params  |  Argument | Type | Description
-----------  | ----------- | -------- | ----------
`getWifiStats`  | Required | Callback | This method returns a Promise that resolves the WiFi stats (stringified).

```javascript
callstats.attachWifiStatsHandler(getWifiStats);

function getWifiStats() {
  return new Promise(function(resolve, reject) {
    resolve(JSON.stringify(wifistats));
  });
}

wifistats = {
  signal: 20, // mandatory
  rssi: -1, // mandatory
  timestamp: 152320980808, // mandatory
  interface: 'en0', // optional
  noise: -90,  // optional
  addresses: ['192.168.0.1', '192.168.0.2'],  // optional
}

```

## API return values

- All the callstats.io APIs returns an object containing `status` and `msg`.

   Params   | Description
----------- | ----------
`status`   | Returns one of the values defined in [callStatsAPIReturnStatus](#enumeration-of-callstatsapireturnstatus)
`msg`   | Returns the corresponding message related to the `status`.


## callstats.startPrecallTests()

```javascript
callstats.startPrecallTests(iceServers, interval);

var iceServers = [
   {
      "urls":[
         "turns:taas.callstats.io:443?transport=udp",
         "turns:taas.callstats.io:443?transport=tcp",
         "turn:taas.callstats.io:80?transport=udp",
         "turn:taas.callstats.io:80?transport=tcp"
      ],
      "username":,
      "credential":,
      "label": "turn2"
   },
   {
      "urls":[
        "turn:turn-server-1.dialogue.io:3478"
      ],
      "username":,
      "credential":,
      "label": "turn2"
    }
];

```
- This API can be used to make precall tests continuously as per the interval given by the customers. The results of the precall test will be given in the `preCallTestResults` callback. Once we run precall tests on all the turn credentials provided, the connection recommendation is provided in the `connectionRecommendation` callback.


  Params  |  Argument | Type | Description
-----------  | ----------- | -------- | ----------
`iceServers`  | Required | Object | TURN Server list provided by customer.
`interval`  | Required | Integer | Indicates the delay (in seconds) after the tests to restart the pre call tests.

<aside class="error">
<ul>

<li> Interval default value is 60 seconds, if not provided. The minimum interval is 30 seconds, and the recommended interval is 300 seconds</li>

</ul>
</aside>


## callstats.stopPrecallTests()
```javascript
callstats.stopPrecallTests();
```
- This API to stop the precall tests started using the API `startPrecallTests()`

## callstats.sendCallDetails()

```javascript
callstats.sendCallDetails(pc, conferenceID, callAttributes);
var callAttributes = {
  callType:,  //inbound or outbound or monitoring
  role:       //agent or participant or manager
}
```
- This API reports the call details from Contact Centers to Callstats backend.

 Params  |  Argument | Type | Description
-----------  | ----------- | -------- | ----------
`pc`  | Required | Object | RTCPeerConnection associated with this connection.
`conferenceID`  | Required | String (256 bytes) | It is generated by the origin server.
`callAttributes`  | Required | JSON | Contains information about the call.
