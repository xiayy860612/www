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

## Endpoints

### IDP endpoints

- discovery endpoint(`/.well-known/openid-configuration`)
- auth endpoint(`/auth`)
- token endpoint(`/token`)
- JWKS endpoint(`/.well-known/jwks.json`)
- user endpoint(`/me`)
- end session endpoint(`/session/end`)

### Client endpoints

- login callback
- logout callback

## authorization_code Workflow

```puml

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

for SPA application without server, it recommends use `authorization_code + PCKE` flow.

## Token

- id token
- access token
- refresh token

## Reference

- [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0-final.html)
