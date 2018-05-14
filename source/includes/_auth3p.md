# Third-party authentication

Token format used by callstats.io's third party authentication system is [JWT](https://jwt.io/). Currently the only supported algorithm is ES256.

## API

The API is defined [above](#callstats-initialize-with-third-party-authentication). 

## Token claims

 ```json
 {
  "userID":"4358",
  "appID":"545619706",
  "keyID":"0123456789abcedf00",
  "iat":1465221382,
  "nbf":1465221682,
  "exp":1465221682,
  "jti":"25b30fb33a7764d2971534507718f35274bb"
}
```

Token supports following claims:

  Claim  |  Required/optional | Type | Description
-----------  | ----------- | -------- | ---------- 
 appID | Required | String | This is your AppID.
 userID | Required | String | This is endpoint's local user identifier.
 keyID | Required | String | ID of the key that was used to generate this token. This can be obtained from secrets configuration page
 exp | Optional | NumericDate | [Token expiration time](https://tools.ietf.org/html/rfc7519#section-4.1.4). It's recommended that this is set 5-10 minutes into the future.
 nbf | Optional | NumericDate | [Token not valid before](https://tools.ietf.org/html/rfc7519#section-4.1.5). It's recommended that this is set 5-10 minutes into the past.
 jti | Optional | String | [JWT ID](https://tools.ietf.org/html/rfc7519#section-4.1.7). This is used for audit logging.
 
## Alternative Step 2: Initialize() with JWT (Client example)

```javascript
  //initialize the app with application tokens
  var AppID     = "YOUR APPLICATION ID";

  function exampleTokenGenerator(initialToken) {
    var cached = null;
    if (initalToken)
     var cached = initialToken;
    // forcenew = set to true if application should generate new token and false if
    // it's okay to use cached token
    // callback(error, token). error should be set to non-null if there was an
    // non-recoverable error. Token should be the JWT. Please see section
    // "Third-party Authentication" for more complete documentation
    return function(forcenew, callback) {
      if (!forcenew && cached !== null)
        return callback(null, cached);
      // 1. get new token
      var xhr = new XMLHttpRequest();
      xhr.open('POST', '/getToken');
      xhr.setRequestHeader('Content-Type', 'application/json; charset=UTF-8');
      xhr.onload = function() {
        // Did we get 200 OK response?
        if (xhr.status == 200) {
          // Get token and send it to callback
          var resp = JSON.parse(req.responseText);
          // the token should contain the claims defined in third party authentication
          return callback(null, resp.token);
        }
        console.log("Couldn't get the token");
        console.log(req.responseText);
        // if uncorrectable error happens, inform callstats.io
        return callback('Unknown error');
      };
      xhr.send(data);
    };
  }

  function csInitCallback (err, msg) {
    console.log("Initializing Status: err="+err+" msg="+msg);
  }

  //userID is generated or given by the origin server
  callstats.initialize(AppID, exampleTokenGenerator(initialToken), userID, csInitCallback, csStatsCallback, configParams);
```


After the user is authenticated with the origin server (or when the page loads), call `initialize()` with appropriate parameters (see [API section](#api)).  Check the callback for errors.  If the authentication succeeds, `callstats.js` will receive an appropriate authentication token to make subsequent API calls.

## Generating a JSON Web Token (JWT)

```javascript
  var express = require('express');
  var jwt = require('jsonwebtoken');
  var app = express();
  var fs = require('fs');

  var crypto = require('crypto');

  // Your ECDSA private key for which the public key is submitted to
  // callstats.io dashboard
  var key = fs.readFileSync('ecprivate.key');
  var keyid = '0102030405060709';
  var appid = 12345678;
  // Dummy audit log
  var audit = {log: console.log};

  // Dummy session middleware
  app.use(function (req, res, next) {
    req.user = {id: 300};
    next();
  });

  // Dummy JWT ID generator
  var randomIdGenerator = function () {
    return 42;
  };

  app.post('/getToken', function (req, res) {
    if (!req.user) {
      res.status(403);
      return res.send(JSON.stringify({error: "User not logged in"}));
    }
    var randomid = randomIdGenerator().toString();
    var token = null;
    try {
      token = jwt.sign(
        {
          userID: req.user.id.toString(),
          appID: appid,
          keyID: keyid
        }, key,
        {
          algorithm: "ES256",
          jwtid: randomid,
          expiresIn: 300, //5 minutes
          notBefore: -300 //-5 minutes
        });
    } catch (err) {
      console.log(err);
      res.status(500);
      return res.send(JSON.stringify({error: "Token creation failed"}));
    }
    audit.log({action: "GrantToken", user: req.user.id, tokenid: randomid});
    res.status(200);
    res.send(JSON.stringify({token: token}));
  });

  app.listen(3000, function () {
    console.log('Example app listening on port 3000!');
  });
```

You can use the following code as an example to generate JWT for authenticating the endpoint.

To correctly generate a token keep in mind the following:

1. Create a token header as a JSON:
```
{
  "typ":"JWT", 
  "alg":"HS256"
}
```

2. Create a token payload as a JSON in the format: 
```
{ 
  "userID": x, 
  "appID": a, 
  "keyID": k, 
  "iat": i, 
  "nbf": n, 
  "exp": e, 
  "jti": j 
}
```  
   <br/>
   Please note that `iat` , `nbf`, `exp`, `jti` are optional, but it is recommended to use it, otherwise the token won't be timed out. 
   More informationn can be found in [RFC 7519](https://tools.ietf.org/html/rfc7519).

3. Encode with base64 stringified header and payload and join them with dot. 
   In pseudo code something like:
   <br/>
   `token = base64Encode(JSON.stringify(header)) + "." + base64Encode(JSON.stringify(payload))`

4. Sign your token with HMAC SHA-256 algorithm
5. Use signed token "code" in the message that you send in the authentication request