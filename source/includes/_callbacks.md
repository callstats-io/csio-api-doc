
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
  inbound: 'inbound',
  outbound: 'outbound'
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

```

The csStatsCallback function can either be given as a parameter to the initialize() call, or set separately with the on() functionality.

The `csStatsCallback()` will be called by the callstats.js for each PeerConnection independently at regular intervals. By default the interval is set as 10 seconds to make sure we do not overwhelm the app with too many messages. For more information, please check out our blog on [`csStatsCallback()`] (http://www.callstats.io/2015/08/24/statscallback-webrtc-media-quality-status/)

## The pre-call test results callback

```javascript

//Usage
callstats.on("preCallTestResults", csPreCallTestResultsCallback);

function preCallTestResultsCallback(status, results) {

//Check the status
  if (status == 'success') {
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

The `csPreCallTestResultsCallback` function is set with the on() functionality. The callback is invoked when the pre-call test results are available. The pre-call test measures the media connectivity, Round Trip Time, Fractional Loss, and Throughput against callstats.io TURN servers. You can use “status” to check if the pre-call test is running, it will return `success` or `failure`. The pre-call test is running as long as the callback is not fired. The pre-call test might return partial results if the tests are interrupted or the call begins before the pre-call test is completed. You can disable pre-call test by adding "disablePrecalltest" in `configParams`.

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

## The error callback

```javascript

//Usage
callstats.on("error", csErrorCallback);

var csErrorCallback(errorType, eventMsg) {
// We have one errorType which is oneWayMedia
// eventMsg Object contains mediaType, SSRC, disruptionType
// mediaType ['audio', 'video']
// disruptionType
var _disruptionsType = {
noAudioInMultiplexFabric: 'noAudioInMultiplexFabric',
noOutboundAudioOnlyFabric: 'noOutboundAudioOnlyFabric',
noInboundAudioOnlyFabric: 'noInboundAudioOnlyFabric'
};
}
```

The csErrorCallback function is set with the on() functionality. The callback can be invoked anytime and returns the error type and event message. The event message contains the attributes for that error type.
