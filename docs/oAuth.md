# oAuth 2.0

OAuth is a delegated authorization framework for REST/APIs. It enables apps to obtain limited access (scopes) to a user’s data without giving away a user’s password. It decouples authentication from authorization and supports multiple use cases addressing different device capabilities. It supports server-to-server apps, browser-based apps, mobile/native apps, and consoles/TVs.

You can think of this like hotel key cards, but for apps. If you have a hotel key card, you can get access to your room. How do you get a hotel key card? You have to do an authentication process at the front desk to get it. After authenticating and obtaining the key card, you can access resources across the hotel.

To break it down simply, OAuth is where:

- App requests authorization from User
- User authorizes App and delivers proof
- App presents proof of authorization to server to get a Token
- Token is restricted to only access what the User authorized for the specific App

> oAuth 2.0 is not an authentication protocol 

## Which oAuth 2.0 Flow Should I Use?

- Web application with server backend: **Authorization Code** flow
- Native mobile app: **Authorization Code flow with PKCE** or use **AppAuth**
- Microservices and APIs: **Client Credentials** flow

## To Obtain Authorization

1. Register your application via #api-dev
2. Follow Xendit authorization flow

## Authorization Code Flow

![Authorization flow](../assets/images/authorization_code_flow.png)

### 1. Have your application request authorization; the user logs in and authorize access

Your application sends a request to the Xendit service. The reason your application sends this request may vary:

- A step in the initialization of your application
- A response to a user action, like a button click "Login with Xendit"

![Xendit Consent Page](../assets/images/xendit-consent-page.png)

The `GET` request is sent to the `/auth` endpoint of the oAuth service

```json http
{
  "method": "get",
  "url": "https://dashboard.xendit.co/auth",
  "query": {
    "client_id": "b9deb6f6-63bd-46bb-823e-dd6ee7200fe8",
    "redirect_uri": "https://webhook.xendit.co",
    "response_type": "code",
    "scopes": "VIRTUAL_ACCOUNT.READ",
    "state": "b9deb6f6-63bd-46bb"
  }
}
```

Responses

<!--
type: tab
title: 200
-->

```json
{
  "code": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImp0aSI6ImUzMTU5ZjIyLTI1OTQtNGNiYS1hMDMzLTRhNWY4MGQ1MmQwZiIsImlhdCI6MTU4OTg3NjI4MywiZXhwIjoxNTg5ODc5ODgzfQ.H4DoU_zLXSUvzkSKIdYR3cXhwoQ-9gqkg491DsyTzg4",
  "state": "b9deb6f6-63bd-46bb"
}
```

<!--
type: tab
title: 403
-->

```json
{
  "error": "access_denied",
  "state": "b9deb6f6-63bd-46bb"
}
```

<!-- type: tab-end -->

### 2. Have your application exchange authorization code for refresh and access tokens; Xendit returns acceess and refresh tokens

When the authorization code has been received, you will need to exchange it with an access token by making a POST request to the Xendit APIs

The request is sent to token endpoint of the Xendit APIs:
`POST https://api.xendit.co/oauth/tokens`

The body of this POST request must contain the following parameters:

| Parameters    | Data Type | Required? | Description                                                     |
| ------------- | --------- | --------- | --------------------------------------------------------------- |
| grant_type    | string    | Required  | Set to `authorization_code`                                          |
| code | string    | Required  | Authorization code retrieved when user granted their consent to the application |
| redirect_uri | string    | Required  | The URI to redirect to after the user grants or denies permission. This URI needs to have been entered in the Redirect URI whitelist that you specified when you registered your application. The value of redirect_uri here must exactly match one of the values you entered when you registered your application, including upper or lowercase, terminating slashes, and such |
| client_id     | string    | Required  | oAuth client ID for your application                            |
| client_secret | string    | Required  | oAuth client secret for you application                         |

```json http
{
  "method": "post",
  "url": "/oauth/tokens",
  "headers": {
    "Content-Type": "application/json"
  },
  "body": {
    "grant_type": "authorization_code",
    "code": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImp0aSI6ImUzMTU5ZjIyLTI1OTQtNGNiYS1hMDMzLTRhNWY4MGQ1MmQwZiIsImlhdCI6MTU4OTg3NjI4MywiZXhwIjoxNTg5ODc5ODgzfQ.H4DoU_zLXSUvzkSKIdYR3cXhwoQ-9gqkg491DsyTzg4",
    "redirect_uri": "https://sample.application.com/payments",
    "client_id": "b9deb6f6-63bd-46bb-823e-dd6ee7200fe8",
    "client_secret": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJidXNpbmVzc19pZCI6IjViOGY3ZDg2M2JjNzllOTA3ZmU0OTA1MCIsImNsaWVudF9pZCI6IjFkY2MyNDIwLTY5YjktNGE0YS05MWFmLWZlYTAwYzY2OThlNyIsImVudmlyb25tZW50IjoiREVWRUxPUE1FTlQiLCJpYXQiOjE1ODkyNzE2NTgsImV4cCI6NDcxMzQ3NDA1OH0.N4XSx2CmKkH0OahAn4-MUyK6UA4o80USUVbqJkk2jrE"
  }
}
```

Responses

<!--
type: tab
title: 200
-->

```json
{
  "token_type": "Bearer",
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJidXNpbmVzc19pZCI6IjViOGY3ZDg2M2JjNzllOTA3ZmU0OTA1MCIsImNsaWVudF9pZCI6IjFkY2MyNDIwLTY5YjktNGE0YS05MWFmLWZlYTAwYzY2OThlNyIsImVudmlyb25tZW50IjoiREVWRUxPUE1FTlQiLCJpYXQiOjE1ODkyNzE2NTgsImV4cCI6NDcxMzQ3NDA1OH0.N4XSx2CmKkH0OahAn4-MUyK6UA4o80USUVbqJkk2jrE",
  "scope": "virtual_accounts.write ewallets.write",
  "expires_at": "2020-02-20T16:59:20Z",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJidXNpbmVzc19pZCI6IjViOGY3ZDg2M2JjNzllOTA3ZmU0OTA1MCIsImNsaWVudF9pZCI6IjFkY2MyNDIwLTY5YjktNGE0YS05MWFmLWZlYTAwYzY2OThlNyIsInNjb3BlcyI6WyJJTlZPSUNFLlJFQUQiLCJJTlZPSUNFLldSSVRFIl0sImVudmlyb25tZW50IjoiREVWRUxPUE1FTlQiLCJpYXQiOjE1ODkyODEzODgsImV4cCI6MTU4OTI4MTM4OH0.nw6MdpfdVy3Q9G7r5X2iTxBV59NeS8l0J--P0b4mHrg"
}
```

<!--
type: tab
title: 403
-->

```json
{
  "error_code": "INVALID_SCOPE_ERROR",
  "message": "The requested scopes are not authorized for your request. Please check the scopes full list in ... or contact Xendit if you believe the error is incorrect",
  "links": {
    "href": "https.docs.xendit.co",
    "rel": "docs"
  }
}
```

<!--
type: tab
title: 404
-->

```json
{
  "error_code": "DATA_NOT_FOUND",
  "message": "We cannot find the client ID in our system. Please sign up your application by contacting us via luminto@xendit.co or re-check if you have attached the correct client ID in the request"
}
```

<!-- type: tab-end -->

### 3. Use the access token to access Xendit APIs; Xendit returns requested data

The access token allows you to make requests to the Xendit API on behalf of a user, for example

`curl -H "Authorization: Bearer NgCXRK...MzxYjw" https://api.xendit.co/callback_virtual_accounts`

```json
{
  "external_id": "PAYM-12394890198023",
  "bank_code": "BNI",
  "name": "Luminto"
}
```

### 4. Requesting a refreshed access token; Xendit returns a new access token to your application

Access tokens are deliberately set to expire after a short time, after which new tokens may be granted by supplying the refresh token originally obtained during the authorization code exchange.

The request is sent to token endpoint of the Xendit APIs:
`POST https://api.xendit.co/oauth/tokens`

The body of this POST request must contain the following parameters:

| Parameters    | Data Type | Required? | Description                                                     |
| ------------- | --------- | --------- | --------------------------------------------------------------- |
| grant_type    | string    | Required  | Set to `refresh_token`                                          |
| refresh_token | string    | Required  | The refresh token returned from the authorization code exchange |
| client_id     | string    | Required  | oAuth client ID for your application                            |
| client_secret | string    | Required  | oAuth client secret for you application                         |

```json http
{
  "method": "post",
  "url": "/oauth/tokens",
  "headers": {
    "Content-Type": "application/json"
  },
  "body": {
    "grant_type": "refresh_token",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJidXNpbmVzc19pZCI6IjViOGY3ZDg2M2JjNzllOTA3ZmU0OTA1MCIsImNsaWVudF9pZCI6IjFkY2MyNDIwLTY5YjktNGE0YS05MWFmLWZlYTAwYzY2OThlNyIsInNjb3BlcyI6WyJJTlZPSUNFLlJFQUQiLCJJTlZPSUNFLldSSVRFIl0sImVudmlyb25tZW50IjoiREVWRUxPUE1FTlQiLCJpYXQiOjE1ODkyODEzODgsImV4cCI6MTU4OTI4MTM4OH0.nw6MdpfdVy3Q9G7r5X2iTxBV59NeS8l0J--P0b4mHrg",
    "client_id": "b9deb6f6-63bd-46bb-823e-dd6ee7200fe8",
    "client_secret": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJidXNpbmVzc19pZCI6IjViOGY3ZDg2M2JjNzllOTA3ZmU0OTA1MCIsImNsaWVudF9pZCI6IjFkY2MyNDIwLTY5YjktNGE0YS05MWFmLWZlYTAwYzY2OThlNyIsImVudmlyb25tZW50IjoiREVWRUxPUE1FTlQiLCJpYXQiOjE1ODkyNzE2NTgsImV4cCI6NDcxMzQ3NDA1OH0.N4XSx2CmKkH0OahAn4-MUyK6UA4o80USUVbqJkk2jrE"
  }
}
```
Responses

<!--
type: tab
title: 200
-->

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJidXNpbmVzc19pZCI6IjViOGY3ZDg2M2JjNzllOTA3ZmU0OTA1MCIsImNsaWVudF9pZCI6IjFkY2MyNDIwLTY5YjktNGE0YS05MWFmLWZlYTAwYzY2OThlNyIsImVudmlyb25tZW50IjoiREVWRUxPUE1FTlQiLCJpYXQiOjE1ODkyNzE2NTgsImV4cCI6NDcxMzQ3NDA1OH0.N4XSx2CmKkH0OahAn4-MUyK6UA4o80USUVbqJkk2jrE",
  "token_type": "Bearer",
  "scope": "virtual_account.write ewallets.write",
  "expires_at": "2020-02-20T16:29:51Z"
}
```
<!-- type: tab-end -->
