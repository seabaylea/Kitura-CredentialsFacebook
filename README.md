<p align="center">
    <a href="http://kitura.io/">
        <img src="https://raw.githubusercontent.com/IBM-Swift/Kitura/master/Sources/Kitura/resources/kitura-bird.svg?sanitize=true" height="100" alt="Kitura">
    </a>
</p>


<p align="center">
    <a href="http://www.kitura.io/">
    <img src="https://img.shields.io/badge/docs-kitura.io-1FBCE4.svg" alt="Docs">
    </a>
    <a href="https://travis-ci.org/IBM-Swift/Kitura-CredentialsFacebook">
    <img src="https://travis-ci.org/IBM-Swift/Kitura-CredentialsFacebook.svg?branch=master" alt="Build Status - Master">
    </a>
    <img src="https://img.shields.io/badge/os-macOS-green.svg?style=flat" alt="macOS">
    <img src="https://img.shields.io/badge/os-linux-green.svg?style=flat" alt="Linux">
    <img src="https://img.shields.io/badge/license-Apache2-blue.svg?style=flat" alt="Apache 2">
    <a href="http://swift-at-ibm-slack.mybluemix.net/">
    <img src="http://swift-at-ibm-slack.mybluemix.net/badge.svg" alt="Slack Status">
    </a>
</p>

# Kitura-CredentialsFacebook

Plugins for the Credentials framework that authenticate using Facebook

## Summary
Plugins for [Kitura-Credentials](https://github.com/IBM-Swift/Kitura-Credentials) framework that authenticate using the [Facebook web login with OAuth](https://developers.facebook.com/docs/facebook-login/manually-build-a-login-flow) and a [Facebook OAuth token](https://developers.facebook.com/docs/facebook-login/access-tokens) that was acquired by a mobile app or other client of the Kitura based backend.

## Table of Contents
* [Swift version](#swift-version)
* [Example of Facebook web login](#example-of-facebook-web-login)
* [Example of authentication with Facebook OAuth token](#example-of-authentication-with-facebook-oauth-token)
* [License](#license)

## Swift version
The latest version of Kitura-CredentialsFacebook requires **Swift 4.0** or newer. You can download this version of the Swift binaries by following this [link](https://swift.org/download/). Compatibility with other Swift versions is not guaranteed.

## Example of Facebook web login
A complete sample can be found in [Kitura-Credentials-Sample](https://github.com/IBM-Swift/Kitura-Credentials-Sample).
<br>

First set up the session:

```swift
import KituraSession

router.all(middleware: Session(secret: "Very very secret..."))
```
Create an instance of `CredentialsFacebook` plugin and register it with `Credentials` framework:

```swift
import Credentials
import CredentialsFacebook

let credentials = Credentials()
let fbCredentials = CredentialsFacebook(clientId: fbClientId,
                                        clientSecret: fbClientSecret,
                                        callbackUrl: serverUrl + "/login/facebook/callback",
                                        options: options)
credentials.register(fbCredentials)
```

**Where:**
   - *fbClientId* is the App ID of your app in the Facebook Developer dashboard
   - *fbClientSecret* is the App Secret of your app in the Facebook Developer dashboard
   - *options* is an optional dictionary ([String:Any]) of Facebook authentication options whose keys are listed in `CredentialsFacebookOptions`.

**Note:** The *callbackUrl* parameter above is used to tell the Facebook web login page where the user's browser should be redirected when the login is successful. It should be a URL handled by the server you are writing.
Specify where to redirect non-authenticated requests:
```swift
credentials.options["failureRedirect"] = "/login/facebook"
```

Connect `credentials` middleware to requests to `/private`:

```swift
router.all("/private", middleware: credentials)
router.get("/private/data", handler:
    { request, response, next in
        ...  
        next()
})
```
And call `authenticate` to login with Facebook and to handle the redirect (callback) from the Facebook login web page after a successful login:

```swift
router.get("/login/facebook",
           handler: credentials.authenticate(fbCredentials.name))

router.get("/login/facebook/callback",
           handler: credentials.authenticate(fbCredentials.name))
```

## Example of authentication with Facebook OAuth token

This example shows how to use `CredentialsFacebookToken` plugin to authenticate post requests, it shows both the server side and the client side of the request involved.

### Server side

First create an instance of `Credentials` and an instance of `CredentialsFacebookToken` plugin:

```swift
import Credentials
import CredentialsFacebook

let credentials = Credentials()
let fbCredentials = CredentialsFacebookToken(options: options)
```
**Where:**
- *options* is an optional dictionary ([String:Any]) of Facebook authentication options whose keys are listed in `CredentialsFacebookOptions`.

Now register the plugin:

```swift
credentials.register(fbCredentials)
```

Connect `credentials` middleware to post requests:

```swift
router.post("/collection/:new", middleware: credentials)
```
If the authentication is successful, `request.userProfile` will contain user profile information received from Facebook:

```swift
router.post("/collection/:new") {request, response, next in
  ...
  let profile = request.userProfile
  let userId = profile.id
  let userName = profile.displayName
  ...
  next()
}
```

### Client side
The client needs to put [Facebook access token](https://developers.facebook.com/docs/facebook-login/access-tokens) in request's `access_token` HTTP header field, and "FacebookToken" in `X-token-type` field:

```swift
let urlRequest = NSMutableURLRequest(URL: NSURL(string: "http://\(serverUrl)/collection/\(name)"))
urlRequest.HTTPMethod = "POST"
urlRequest.HTTPBody = ...

urlRequest.addValue(FBSDKAccessToken.currentAccessToken().tokenString, forHTTPHeaderField: "access_token")
urlRequest.addValue("FacebookToken", forHTTPHeaderField: "X-token-type")            

Alamofire.request(urlRequest).responseJSON {response in
  ...
}

```
## License
This library is licensed under Apache 2.0. Full license text is available in [LICENSE](LICENSE.txt).
