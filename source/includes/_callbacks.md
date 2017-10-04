
# Callbacks and Error Handling

The WebRTC application can provide callback functions to callstats.js which are invoked in case of the following events:

- After the initialize
- To enquire the stats
- To obtain the default configuration


## The initialize callback

```javascript
function csInitCallback(csError, csErrMsg) {
  console.log("Status: errCode= " + csError + " errMsg= " + csErrMsg ); }
}
```

The csInitCallback function is given as a parameter to the `initialize()` call. To report different success and failure cases, they can occur during `initialize()` or sending measurements to callstats.io. The callback takes the form of:

csError and csErrMsg are of type _String_. `csErrMsg` is a descriptive error returned by callstats.io.

## The stats callback

```javascript
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

// initialize the callstats js API
var callstats = new callstats();
callstats.initialize(AppID, AppSecret, localUserId, csInitCallback, csStatsCallback);
```

The csStatsCallback function can either be given as a parameter to the initialize() call, or set separately with the on() functionality. The `initialize()` API authenticates the javascript WebRTC application with the callstats.io back-end, and sets up a trusted relationship with it. The API is extended by adding a `csStatsCallback` parameter. The callback parameter is OPTIONAL.

The `csStatsCallback()` will be called by the callstats.js for each PeerConnection independently at regular intervals. By default the interval is set as 10 seconds to make sure we do not overwhelm the app with too many messages. For more information, please check out our blog on [`csStatsCallback()`] (http://www.callstats.io/2015/08/24/statscallback-webrtc-media-quality-status/)

## The default configuration callback

```javascript
function csDefaultConfigurationCallback(config) {
  var pc = new RTCPeerConnection(config.peerConnection);
  getUserMedia(config.media);

}
```

The csDefaultConfigurationCallback function is set with the on() functionality. The callback is invoked when the default configuration provided in the callstats.io dashboard is available. It is only fired once.

<aside class="error">
<ul>

<li> While we maintain high reliability and availability in our service, we still recommend that the app should not crash or block on this callback</li>
<li> The default configuration provided in the callback is the exact entry you provided on the dashboard. Please make sure that the config is valid and usable by a RTCPeerConnection. This configuration SHOULD be updated with any on-the-fly calculated values in the app </li>

</ul>
</aside>
