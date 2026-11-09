# Spring Security & JWT — Learning Notes

JWT is a token format, not a complete authentication or authorization system. OAuth 2.0 defines authorization flows; OpenID Connect adds an identity layer.

## Two separate responsibilities

- An **authorization server / identity provider** authenticates a user or client and issues access tokens.
- A **resource server** accepts the intended access token, validates it and applies authorization rules to protected APIs.

For a modern Spring MVC API, prefer Spring Security's OAuth2 Resource Server support over a hand-written JWT filter.

## Resource-server request flow

~~~text
Client -> Authorization: Bearer <access-token>
       -> BearerTokenAuthenticationFilter
       -> AuthenticationManager
       -> JwtAuthenticationProvider
       -> JwtDecoder + trusted JWKS public key
       -> validate signature, issuer, timestamps, audience
       -> JwtAuthenticationConverter
       -> SecurityContext
       -> request / method authorization
~~~

## JWT should contain

Only claims required by the application, such as:
- stable subject identifier
- issuer and audience
- issued-at and expiry
- scopes/authorities when appropriate

A signed JWT payload is normally readable; signing does not encrypt it. Do not place passwords, signing secrets or unnecessary sensitive information in claims.

## Security checklist

- Use HTTPS.
- Validate signature with trusted keys and an explicit allowed algorithm.
- Validate issuer, expiry/not-before and the API's expected audience.
- Use short-lived access tokens where appropriate.
- Keep refresh-token handling at the authorization server; business APIs should accept access tokens.
- Protect signing keys and support safe JWKS key rotation.
- Apply scope/role checks and object-level authorization at the correct boundary.
- Never log raw access tokens.
- Disable CSRF only for a genuinely stateless Authorization-header API; cookie/session flows need separate CSRF analysis.

## Practical implementation

See the [Spring Security OAuth2 Resource Server implementation guide](OAUTH2_RESOURCE_SERVER_IMPLEMENTATION.md) for Maven dependencies, application.yml, SecurityFilterChain, scope mapping, 401 vs 403, multi-tenant ownership checks, MockMvc tests, real-token validation tests, trade-offs and interview questions.

This is a security learning module. It contains no real credentials or production secrets.
