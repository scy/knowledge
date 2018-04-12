# HTTP

## Bearer Tokens Limited to OAuth?

There's some confusion about whether doing `Authorization: Bearer …` type tokens requires implementing OAuth or whether any kind of token-based authentication qualifies for this scheme. 
Do you have to roll your own scheme (`Authorization: AcmeToken …`) instead?

Funny enough: 
[RFC 6750](https://tools.ietf.org/html/rfc6750) (OAuth 2.0 Bearer Token Usage) answers the question already in its introduction.

> While designed for use with access tokens resulting from OAuth 2.0 authorization [RFC6749] flows to access OAuth protected resources, this specification actually defines a general HTTP authorization method that can be used with bearer tokens from any source to access any resources protected by those bearer tokens.

So, if you're doing token-based authentication, feel free to use `Bearer`.

## 401 Unauthorized Requires a `WWW-Authenticate` Header

The headline basically says it all: 
When your service responds with a `401` status, [RFC 7235 Section 3.1](https://tools.ietf.org/html/rfc7235#section-3.1) states that it also _"MUST send a WWW-Authenticate header field (Section 4.1) containing at least one challenge applicable to the target resource"_.
