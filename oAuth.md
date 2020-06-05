# oAuth 2.0

oAuth 2.0 is the industry-standard protocol for access delegation or authorization, used as a way for Internet users to grant website or applications access to their information on other websites but without giving them the passwords

## Which oAuth 2.0 Flow Should I Use?

- Web application with server backend: **Authorization Code** flow
- Native mobile app: **Authorization Code flow with PKCE** or use **AppAuth**
- Microservices and APIs: **Client Credentials** flow

## To Obtain Authorization

1. Register your application
2. Follow Xendit authorization flow

### Authorization Code Flow

![Authorization flow](../assets/images/authorization_code_flow.png)

#### 1. Have your application request authorization; the user logs in and authorize access

Your application sends a request to the Xendit service. The reason your application sends this request may vary:

- A step in the initialization of your application
- A response to a user action, like a button click "Login with Xendit"

The `GET` request is sent to the `/auth` endpoint of the oAuth service

```json http
{
  "method": "get",
  "url": "https://dashboard.xendit.co/auth",
  "query": {
    "client_id": "b9deb6f6-63bd-46bb-823e-dd6ee7200fe8",
    "grant_type": "code",
    "scopes": "[\"INVOICE.READ\", \"INVOICE.WRITE\"]"
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
  "code": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImp0aSI6ImUzMTU5ZjIyLTI1OTQtNGNiYS1hMDMzLTRhNWY4MGQ1MmQwZiIsImlhdCI6MTU4OTg3NjI4MywiZXhwIjoxNTg5ODc5ODgzfQ.H4DoU_zLXSUvzkSKIdYR3cXhwoQ-9gqkg491DsyTzg4"
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

#### 2. Have your application exchange authorization code for refresh and access tokens; Xendit returns acceess and refresh tokens

When the authorization code has been received, you will need to exchange it with an access token by making a POST request to the Xendit APIs

```json http
{
  "method": "post",
  "url": "/oauth/tokens",
  "headers": {
    "Content-Type": "application/json"
  },
  "body": {
    "code": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImp0aSI6ImUzMTU5ZjIyLTI1OTQtNGNiYS1hMDMzLTRhNWY4MGQ1MmQwZiIsImlhdCI6MTU4OTg3NjI4MywiZXhwIjoxNTg5ODc5ODgzfQ.H4DoU_zLXSUvzkSKIdYR3cXhwoQ-9gqkg491DsyTzg4",
    "client_secret": ""
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
  "code": "b9deb6f6-63bd-46bb-823e-dd6ee7200fe8"
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

#### 3. Use the access token to access Xendit APIs; Xendit returns requested data

#### 4. Requesting a refreshed access token; Spotify returns a new access token to your application

2. If the user gives authorization, the client passes the authorization grant to the authorization server
3. If the grant is valid, the authorization server returns an access token, possibly alongside a refresh and/or ID token
4. The client now uses that access token to access the resource server
