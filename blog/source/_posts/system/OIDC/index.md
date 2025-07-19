---
title: OIDC Design
date: 2025-07-13 14:00:19
categories:
- system
tags:
- oidc
- oauth2
---

OIDC Infrastructure design

<!--more-->

## OIDC IDP Design

### Endpoints

- discovery endpoint(`/.well-known/openid-configuration`), show all endpoints for client
- auth endpoint(`/auth`), trigger auth workflow
- token endpoint(`/token`), get token info
- JWKS endpoint(`/.well-known/jwks.json`), is used to verify jwt token, such like id token
- user endpoint(`/me`), get lateset user info
- end session endpoint(`/session/end`), is used to logout

### Token

#### ID Token

It's a `JWT token`, represents a user credentials for successful login user,
and includes basic user profile which defined by scope and claim.

It's always passed to client and is used show user info.

most important fields in ID Token:

- sub(Subject), subject contains user unique id
- iss(Issuer), IDP host which signed ID Token
- exp(Expiration time), expiration timestamp for ID Token
- iat(Issued at), sign timestamp for ID Token
- aud(Audience), mostly is client id
- user claims, show basic user info

```json
{
   "iss": "https://idp-doamin",
   "sub": "24400320",
   "aud": "client-id",
   "exp": 1311281970,
   "iat": 1311280970,
   ...
   <user claims>
}
```

Client uses HTTP Header `Authorization: "Bearer <ID Token>"` to send request to server,
then server could use `JWKS endpoint` to get public key to verify ID Token.

#### Access Token

It mostly is an `Opaque token` which is a random string
and used to access IDP resources, such like below cases:

- invoke user endpoint to get more user details.

Access token is mostly `saved in backend`. If client required IDP resources, backend service could be proxy to get related resources and return to client.

#### Refresh Token

It's used to get a new Access Token and ID Token when Access Token is expired by invoking `token endpoint`.

It should be `saved in backend` for security.

It requires scope `offline_access` and `prompt=consent` in querystring when trigger auth workflow.

More details for reference:

- [刷新 Access Token](https://docs.authing.cn/v2/guides/federation/oidc.html#%E5%88%B7%E6%96%B0-access-token)

#### Expiration Time for Tokens

`Refresh Token > ID Token >= Access Token`

When ID Token/Access Token is expired, client can trigger refresh token action in server to get new one,
that's why expiration time for ID Token/Access Token could be short.

When Refresh Token is expired, user must re-login again to obtain new token info include refresh token,
to avoid requiring users to login frequently, refresh tokens are given a long expiration time.

### auth Workflow with authorization_code

```plantuml

actor User as U
actor Client as C
control "Resource Service" as S
control "OIDC IDP" as IDP

U --> C: trigger login
C --> S: apply login url
S --> C: return generated one-time url for auth endpoint

group login phase
C --> IDP: redirect to generated auth endpoint and show login page
U --> IDP: fill user credentials
IDP --> IDP: verify user credentials
end

group consent phase
IDP --> C: show consent page
U --> IDP: confirm in consent page
IDP --> IDP: create session for login user
IDP --> C: redirect login callback
end

group get token phase
C --> S: send received code
S --> IDP: get token info by code with token endpoint
IDP --> S: return token info include id token, access token, refresh token and profile.
S --> IDP: use access token to get user info by user endpoint
S --> S: save user info and token info with session
S --> C: return user info and access token
C --> C: save access token and show user info from id token
C --> C: redirect to root page
end

group get resource phase

U --> C: access resources
C --> S: send request to apply resource with access token
S --> IDP: verify access token
S --> S: get user info from session and check permissions
S --> C: return user related resources

end

group refresh token in slient

C --> S: trigger refresh access token when it will be expired
S --> IDP: get new access token with original access token and refresh token
S --> IDP: use access token to get user info by user endpoint
S --> C: return user info and access token
C --> C: save user info and access token

end

group logout phase

U --> C: trigger logout
C --> S: apply logout url
S --> C: return generated one-time logout url for end session endpoint
C --> IDP: redirect to end session endpoint with logout url
IDP --> IDP: destroy user session
IDP --> C: rediret to logout callback page
C --> C: clear local session info include user info and access token, then redirect to root page

end

```

good resources for reference:

- demo show in [使用 OIDC 授权码模式](https://docs.authing.cn/v2/federation/oidc/authorization-code/?step=0)

#### SPA with authorization_code + PCKE

for SPA application without server, it recommends use `authorization_code + PCKE` flow.

TODO

### Model Design

TODO

## Client Integration

### Login

- login callback

TODO

### Token Refresh

- refresh token
- Silent Authentication

TODO

### Logout

- logout callback

TODO

## Reference

- [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0-final.html)
- [JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519)
