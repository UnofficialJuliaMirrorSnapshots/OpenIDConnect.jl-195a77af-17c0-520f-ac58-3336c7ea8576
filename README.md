# OpenIDConnect

[![Build Status](https://travis-ci.org/tanmaykm/OpenIDConnect.jl.svg?branch=master)](https://travis-ci.org/tanmaykm/OpenIDConnect.jl)
[![Coverage Status](https://coveralls.io/repos/tanmaykm/OpenIDConnect.jl/badge.svg?branch=master)](https://coveralls.io/r/tanmaykm/OpenIDConnect.jl?branch=master)

[OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html) is a simple identity layer on top of the OAuth 2.0 protocol. It enables Clients to verify the identity of the End-User based on the authentication performed by an Authorization Server, as well as to obtain basic profile information about the End-User in an interoperable and REST-like manner.

This is an implementation of OpenID Connect in Julia, with methods implementing the authorization code flow.

# OpenID Connect Context (OIDCCtx)
The OpenID Connect context holds all states for a single OpenID Connect client configuration.

```julia
function OIDCCtx(
    issuer::String,
    redirect_uri::String,
    client_id::String,
    client_secret::String,
    scopes::Vector{String}=DEFAULT_SCOPES;
    verify::Union{Nothing,Bool}=nothing,
    cacrt::Union{Nothing,String,MbedTLS.CRT}=nothing,
    state_timeout_secs::Int=DEFAULT_STATE_TIMEOUT_SECS,
    allowed_skew_secs::Int=DEFAULT_SKEW_SECS,
    key_refresh_secs::Int=DEFAULT_KEY_REFRESH_SECS)
)
```

Parameters:
- `issuer`: Issuer URL, pointing to the OpenID server
- `redirect_uri`: The app URI to which OpenID server must redirect after authorization
- `client_id`, and `client_secret`: Client ID and secret that this context represents
- `scopes`: The scopes to request during authorization (default: openid, profile, email)
- `

Keyword Parameters:
- `verify`: whether to validate the server certificate
- `cacrt`: the CA certificate to use to check the server certificate
- `state_timeout_secs`: seconds for which to keep the state associated with an authorization request (default: 60 seconds), server responses beyond this are rejected as stale
- `allowed_skew_secs`: while validating tokens, seconds to allow to account for time skew between machines (default: 120 seconds)
- `key_refresh_secs`: time interval in which to refresh the JWT signing keys (default: 1hr)

# Error Structures

- `OpenIDConnect.APIError`: Error detected at the client side. Members:
    - `error`: error code or message (String)
- `OpenIDConnect.AuthServerError`: Error returned from the OpenID server (see section 3.1.2.6 of https://openid.net/specs/openid-connect-core-1_0.html)
    - `error`: error code (String)
    - `error_description`: optional error description (String)
    - `error_uri`: optional error URI (String)

# Authorization Code Flow

## Authentication request.

### `flow_request_authorization_code`
Returns a String with the redirect URL. Caller must perform the redirection.
Acceptable optional args as listed in section 3.1.2.1 of specifications (https://openid.net/specs/openid-connect-core-1_0.html)

```julia
function flow_request_authorization_code(
    ctx::OIDCCtx;
    nonce=nothing,
    display=nothing,
    prompt=nothing,
    max_age=nothing,
    ui_locales=nothing,
    id_token_hint=nothing,
    login_hint=nothing,
    acr_values=nothing
)
```

### `flow_get_authorization_code`
Given the params from the redirected response from the authentication request, extract the authorization code.
See sections 3.1.2.5 and 3.1.2.6 of https://openid.net/specs/openid-connect-core-1_0.html.

Returns the authorization code on success.
Returns one of APIError or AuthServerError on failure.

```julia
function flow_get_authorization_code(
    ctx::OIDCCtx,
    query           # name-value pair Dict with query parameters are received from the OpenID server redirect
)
```

## Token Requests

### `flow_get_token`
Token Request. Given the authorization code obtained, invoke the token end point and obtain an id_token, access_token, refresh_token.
See section 3.1.3.1 of https://openid.net/specs/openid-connect-core-1_0.html.

Returns a JSON object containing tokens on success.
Returns a AuthServerError or APIError object on failure.

```julia
function flow_get_token(
    ctx::OIDCCtx,
    code
)
```

### `flow_refresh_token`
Token Refresh. Given the refresh code obtained, invoke the token end point and obtain new tokens.
See section 12 of https://openid.net/specs/openid-connect-core-1_0.html.

Returns a JSON object containing tokens on success.
Returns a AuthServerError or APIError object on failure.

```julia
function flow_refresh_token(
    ctx::OIDCCtx,
    refresh_token
)
```

## Token Validation

### `flow_validate_id_token`
Validate an OIDC token.
Validates both the structure and signature.
See section 3.1.3.7 of https://openid.net/specs/openid-connect-core-1_0.html

```
function flow_validate_id_token(
    ctx::OIDCCtx,
    id_token::Union{JWTs.JWT, String}
)
```

# Examples

An example application built using OpenIDClient with Mux and HTTP is available as a [tool](tools/oidc_standalone.jl). Populate a configuration file following this [template](tools/settings.template) and start the standalone application. Point your browser to it to experience the complete flow.
