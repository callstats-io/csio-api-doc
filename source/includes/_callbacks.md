
# Callbacks and Error Handling

The WebRTC application can provide callback functions to callstats.js which are invoked in case of the following events:

- After the initialize
- To enquire the stats
- To obtain pre-call test results
- To obtain the default configuration
- To obtain the recommended configuration
- To check the errors


## The initialize callback

```javascript
// initialize the callstats js API
var callstats = new callstats();
callstats.initialize(AppID, AppSecret, localUserId, csInitCallback, csStatsCallback);

function csInitCallback(csError, csErrMsg) {
  console.log("Status: errCode= " + csError + " errMsg= " + csErrMsg ); }
}
```

The csInitCallback function is given as a parameter to the `initialize()` call. To report different success and failure cases, which can occur during `initialize()` or sending measurements to callstats.io. The callback takes the form of:

csError and csErrMsg are of type _String_. `csErrMsg` is a descriptive error returned by callstats.io.

## The stats callback

```javascript
// initialize the callstats js API
var callstats = new callstats();
callstats.initialize(AppID, AppSecret, localUserId, csInitCallback, csStatsCallback);

var reportType = {
  inbound: 'inbound-rtp',
  outbound: 'outbound-rtp'
};

// callback function to receive the stats
var csStatsCallback = function (stats) {
  var ssrc;
  for (ssrc in stats.streams) {
    console.log("SSRC is: ", ssrc);
    var dataSsrc = stats.streams[ssrc];
    console.log("SSRC Type ", dataSsrc.reportType);
    if (dataSsrc.reportType === reportType.outbound) {
      console.log("RTT is: ", dataSsrc.rtt);
    } else if (dataSsrc.reportType === reportType.inbound) {
      console.log("Inbound loss rate is: ", dataSsrc.fractionLoss);
    }
  }
}


// JSON description of the metrics structure

"stats": {
  "connectionState":, // one of: online, offline
  "fabricState":, // one of: initialising, established, disrupted.
  "conferenceURL":, // URL of the conference 

  // New csStatsCallback format

  "mediaStreamTracks": [
    [0]: {
      "remoteUserID":, // associated during the addNewFabric() API
      "statsType":,   // inbound-rtp, outbound-rtp, remote-inbound-rtp, remote-outbound-rtp

      // data here is parsed from the local and remote SDP
      "cname":,      // CNAME associated with the SSRC
      "msid":,       // mediastreamID associated with the SSRC
      "mediaType":,  // either video or audio

      // provided in associateMstWithUserID() API
      "usageLabel":,         //usage information
      "associatedVideoTag":, // video tag that renders the media


      "bitrate":,             // interval bitrate in kbps
      "packetRate":,          // packets per second
      "fractionLoss":,        // interval fractional loss
      "packetLossPercentage":,// Packet loss in percentage   
      "jitter":,              // interval jitter in ms
      "averageJitter":,       // Average jitter
      "rtt":,                 // round-trip time in ms.
      "averageRTT":,          // Average round-trip time in ms
      "quality":,             // interval quality as per boundary conditions.
      "audioInputLevel":,     // audio input level
      "audioOutputLevel":,    // audio output level

    },
  ...
  ]
};
```

The csStatsCallback function can either be given as a parameter to the initialize() call, or set separately with the on() functionality.

The `csStatsCallback()` will be called by the callstats.js for each PeerConnection independently at regular intervals. By default the interval is set as 10 seconds to make sure we do not overwhelm the app with too many messages. For more information, please check out our blog on [`csStatsCallback()`] (http://www.callstats.io/2015/08/24/statscallback-webrtc-media-quality-status/) 

## The pre-call test results callback

```javascript

//Usage
callstats.on("preCallTestResults", csPreCallTestResultsCallback);

function preCallTestResultsCallback(status, results) {

//Check the status
  if (status == callstats.callStatsAPIReturnStatus.success) {
    //Results
    var connectivity = results.mediaConnectivity;
    var rtt = results.rtt;
    var loss = results.fractionalLoss;
    var throughput = results.throughput;

  }
  else {
    console.log("Pre-call test could not be run");
  }

}
```

The `csPreCallTestResultsCallback` function is set with the on() functionality. The callback is invoked when the pre-call test results are available. The pre-call test measures the Media Connectivity, Round Trip Time, Fractional Loss, and Throughput against callstats.io TURN servers. You can use the “status” to check if the pre-call test has succeeded, it will return `success` or `failure`. The pre-call test is enabled by default and is running as long as the callback is not fired. The pre-call test might return partial results if the tests are interrupted or the call begins during the pre-call test. You can disable pre-call test by setting "disablePrecalltest" to `true` in `configParams`.

Params  | Type | Description
-----------  | -------- | ----------
`mediaConnectivity`  | boolean | True or False.
`rtt`  | float | Round Trip Time in ms. Returns "null" if there is no result.
`fractionalLoss`   | float | Fractional Loss [0-1]. Returns "null" if there is no result.
`throughput`  | float| Throughput in kbps. Returns "null" if there is no result.

## The default configuration callback

```javascript
function csDefaultConfigurationCallback(config) {
  var pc = new RTCPeerConnection(config.peerConnection);
  getUserMedia(config.media);
}
```

The `csDefaultConfigurationCallback` function is set with the on() functionality. The callback is invoked when the default configuration provided in the callstats.io dashboard is available. It is only fired once.

- `config.peerConnection` object is of type [RTCConfiguration](https://www.w3.org/TR/webrtc/#rtcconfiguration-dictionary)
- `config.media` object is of type [MediaStreamConstraints](https://www.w3.org/TR/mediacapture-streams/#mediastreamconstraints)

The integration steps are listed [here](#step-10-optional-obtaining-the-default-configuration)

<aside class="error">
<ul>

<li> While we maintain high reliability and availability in our service, we still recommend that the app should not crash or block on this callback</li>
<li> The default configuration provided in the callback is the exact entry you provided on the dashboard. Please make sure that the config is valid and usable by a RTCPeerConnection. This configuration SHOULD be updated with any on-the-fly calculated values in the app </li>

</ul>
</aside>

## The recommended configuration callback

```javascript
function csRecommendedConfigurationCallback(config) {
  var pc = new RTCPeerConnection(config.peerConnection);
  getUserMedia(config.media);
}
```

The `csRecommendedConfigurationCallback` function is set with the on() functionality. The callback is invoked when the recommended configuration provided by callstats.io is available.

<aside class="error">
<ul>

<li> While we maintain high reliability and availability in our service, we still recommend that the app should not crash or block on this callback</li>
<li> The recommended configuration is provided by callstats.io. This configuration SHOULD be updated with any on-the-fly calculated values in the app </li>

</ul>
</aside>

## The connection recommendation callback

```javascript
//Usage
callstats.on("connectionRecommendation", csConnectionRecommendationCallback);

function csConnectionRecommendationCallback(){
   "aggregatedStats":[
      {
         "provider":"turn1",
         "roundTripTime":46.3499755859375,
         "jitter":143.7728006389737,
         "fractionLost":0,
         "throughput":4160.657588552875
      },
      {
         "provider":"turn2",
         "roundTripTime":106.699951171875,
         "jitter":1547.3617393722136,
         "fractionLost":0.5257732840987372,
         "throughput":584.6231721117872
      }
   ],
   "providerRanking":[
      {
         "provider":"turn1",
         "acceptable":true
      },
      {
         "provider":"turn2",
         "acceptable":false
      }
   ]
}
```

The `csConnectionRecommendationCallback` function is set with the on() functionality. The callback is invoked when the precall tests are finished for all the TURN credentials given in 'startPrecallTest'. Connection recommendation gives the statistics measured, and the ranking of the TURN servers provided. 

Params  | Type | Description
-----------  | -------- | ----------
`provider`  | string | TURN label.
`roundTripTime`  | float | Round Trip Time in ms. Returns "null" if there is no result.
`jitter` | float | jitter in ms
`fractionLost`   | float | Fractional Loss [0-1]. Returns "null" if there is no result.
`throughput`  | float| Throughput in kbps. Returns "null" if there is no result.
`acceptable` | boolean | True or False