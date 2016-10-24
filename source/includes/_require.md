# Loading with require.js

> Script loading with require.js

```javascript

<script data-main="YOUR REQUIRE JS CONFIG FILE" src="https://cdnjs.cloudflare.com/ajax/libs/require.js/2.1.15/require.js"></script>

requirejs.config({
  // 1. define source paths to other dependencies
  paths: {
    //jQuery usage is deprecated from the version 3.10.0 onwards and socket.io is deprecated from 3.16.0
    callstats: "https://api.callstats.io/static/callstats.min"
  },
  shim: {
    'callstats': {
      deps: [],
    },
    'WebRTCApp': {
      deps: ['callstats',... <other dependencies> ...],
    }
  }
});

// 3. load your main webRTCApp which depends on callstats
require(['WebRTCApp']);

```


The [require.js](http://www.requirejs.org/) helps load dependencies at the origin server. Below is an example snippet:




