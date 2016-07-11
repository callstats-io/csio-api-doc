
# Authentication

## Basic Authentication over HTTPS

The application requires an `AppID` and `AppSecret` to authenticate with callstats.io. The origin server is expected to pass the `userID` for each endpoint in a WebRTC call. The `callstats.js` internally implements a 4-way handshake, based on simple [challenge-response](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication) protocol. If successful, the callstats.js generates a token valid for 2 hours. The `token` is  subsequently used by the callstats.js to send event and measurement data to callstats.io.

### White-listing (Optional but highly recommended)

callstats.io uses the “Origin” header in the HTTP request to fetch the request’s origin. [RFC6454](http://tools.ietf.org/html/rfc6454#section-4) explains the algorithm used by user-agents to compute the “Origin” header.

callstats.io compares the Origin URL sent by the HTTP user-agent in the authentication message with the stored Origin URL for that particular AppID. If the origins match, callstats.io returns a new token and associates the token to the requesting userID. Alternatively, if the origins does not match, callstats.io rejects the request, and denies access to that particular user-agent.

Instead of relying only on the endpoint for authentication, the callstats.io also implements third-party authentication, which requires the origin server to generate token for the endpoint which is then used to authenticate the endpoint.

Token format used by callstats.io's third party authentication system is [JWT](https://jwt.io/). Currently the only supported algorithm is ES256.

### Token claims

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
