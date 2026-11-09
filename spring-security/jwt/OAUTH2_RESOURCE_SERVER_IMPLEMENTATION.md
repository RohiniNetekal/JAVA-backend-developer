# Spring Security OAuth 2.0 Resource Server — JWT Implementation

> Independent learning/demo material. This guide is not a claim that this exact configuration has been deployed in an employer environment. It uses Spring MVC (Servlet stack), Spring Boot, Java 17+ and an external identity provider / authorization server.

## 1. What are we building?

A resource server protects an API by accepting an OAuth 2.0 **access token** issued by a trusted authorization server.

Keep these terms separate:

- **OAuth 2.0** is an authorization framework.
- **OpenID Connect (OIDC)** adds an identity layer on top of OAuth 2.0.
- **JWT** is a token format. A JWT can be used outside OAuth 2.0 as well.
- **Authorization server / identity provider** authenticates a user or client and issues tokens.
- **Resource server** validates access tokens and enforces access to its APIs.

The API in this exercise does not implement a login page or mint tokens. It validates tokens issued by a separate trusted system. This is normally safer than writing a custom JWT filter and inventing a token format.

## 2. Request flow and internal working

```text
User / service
     |
     | authenticate or obtain client-credentials token
     v
Authorization Server / Identity Provider
     |
     | signed access token
     v
Client -----------------------> Merchant API
                                  |
                                  v
                         SecurityFilterChain
                                  |
                         BearerTokenAuthenticationFilter
                                  |
                         AuthenticationManager
                                  |
                         JwtAuthenticationProvider
                                  |
                              JwtDecoder
                                  |
                     verify signature using JWKS
                     validate iss / exp / nbf / aud
                                  |
                     JwtAuthenticationConverter
                                  |
                     create GrantedAuthorities
                                  |
                         SecurityContextHolder
                                  |
                         endpoint authorization
                                  |
                              Controller
```

A typical Servlet request works like this:

1. `BearerTokenAuthenticationFilter` extracts the token from `Authorization: Bearer ...`.
2. The authentication manager delegates to `JwtAuthenticationProvider`.
3. `JwtDecoder` parses the token, selects a trusted public key from the configured JWK Set (JWKS), and verifies the signature.
4. Validators check registered claims such as issuer and timestamps. The API must also validate its expected audience.
5. `JwtAuthenticationConverter` maps token scopes/claims to Spring `GrantedAuthority` values.
6. Spring stores the authenticated principal in the security context for the request.
7. Request matchers and/or method security decide whether the principal may perform the operation.

The `kid` header helps the decoder select a key from the JWKS. When an identity provider rotates signing keys and publishes the new public key, the resource server can use the updated JWKS. Do not hard-code a short-lived copy of a public key if the provider supports JWKS rotation.

## 3. Add the dependency

Use the Spring Boot dependency-management BOM. Do not independently pin Spring Security transitive modules unless there is a tested compatibility reason.

Maven:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>

<!-- Test support for MockMvc JWT authorization tests -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

The Spring Boot starter brings in the Resource Server and JOSE support required for JWT bearer-token decoding and verification.

## 4. Configure issuer, JWKS, audience and algorithm

Example `application.yml`:

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          # Must exactly match the trusted token issuer (iss claim).
          issuer-uri: ${AUTH_ISSUER_URI}

          # Optional when the issuer metadata can discover the JWKS URI.
          # Supplying both avoids metadata discovery while retaining issuer validation.
          jwk-set-uri: ${AUTH_JWK_SET_URI}

          # The token must be intended for this API (aud claim).
          audiences:
            - ${AUTH_AUDIENCE}

          # Make the accepted signature algorithm an explicit trust decision.
          # Change only to an algorithm agreed with the identity provider.
          jws-algorithms: RS256
```

Provide these values through environment-specific configuration or a secrets/configuration platform. They are identifiers and endpoints, not signing secrets, but should still be configured per environment.

- `issuer-uri` identifies who issued the token. It is checked against `iss`.
- `jwk-set-uri` identifies where the provider publishes public verification keys. If omitted, Spring Security can discover it from issuer metadata.
- `audiences` identifies the API that is allowed to consume the token. A valid token for a different API must not automatically be accepted here.
- `jws-algorithms` restricts which signing algorithm the decoder trusts. Do not trust an algorithm merely because an incoming token header asks for it.

If only `issuer-uri` is configured, Spring Security can use provider metadata to discover the JWKS endpoint. If both issuer and JWKS URI are configured, the direct JWKS endpoint can avoid metadata discovery; keep the issuer configured so that `iss` is still validated.

## 5. Define the security filter chain

This example is for an API that accepts bearer tokens in the HTTP Authorization header and does not authenticate browser requests with cookies or a server-side session.

```java
package com.example.merchant.security;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableMethodSecurity
public class SecurityConfiguration {

    @Bean
    SecurityFilterChain apiSecurity(HttpSecurity http) throws Exception {
        http
            // Appropriate only for this stateless Authorization-header API.
            // Do not disable CSRF for browser/session or cookie-based authentication.
            .csrf(csrf -> csrf.disable())

            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS))

            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers(
                    HttpMethod.GET, "/api/v1/merchants/**")
                    .hasAuthority("SCOPE_merchants.read")
                .requestMatchers(
                    HttpMethod.POST, "/api/v1/merchants/**")
                    .hasAuthority("SCOPE_merchants.write")
                .anyRequest().authenticated())

            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(Customizer.withDefaults()));

        return http.build();
    }
}
```

The configuration delegates JWT decoding and validation to Spring Security's Resource Server support. It does not create a hand-written filter or manually Base64-decode the token.

**Important:** CSRF and CORS solve different problems. CORS controls which browser origins may make cross-origin requests; it is not authentication. If a browser sends credentials automatically through cookies, CSRF protection may still be required even if the application also has JWTs.

## 6. Scopes, roles and method-level authorization

For a standard access token containing:

```json
{
  "sub": "user-101",
  "scope": "merchants.read merchants.write",
  "iss": "https://identity.example.com",
  "aud": ["merchant-api"]
}
```

Spring Security's default scope converter maps scopes to authorities with the `SCOPE_` prefix:

- `merchants.read` becomes `SCOPE_merchants.read`
- `merchants.write` becomes `SCOPE_merchants.write`

Therefore this is correct:

```java
@PreAuthorize("hasAuthority('SCOPE_merchants.read')")
public MerchantResponse getMerchant(Long merchantId) {
    // load and map the merchant
}
```

Do not confuse these with roles:

- `hasAuthority("SCOPE_merchants.read")` checks an OAuth scope.
- `hasRole("ADMIN")` checks for `ROLE_ADMIN` by default.

If an identity provider sends a custom claim such as `roles` instead of `scope` / `scp`, configure a `JwtAuthenticationConverter` and `JwtGrantedAuthoritiesConverter` for that provider's documented claim contract. Decide explicitly whether custom roles replace or are combined with scopes. Never assume a claim called `role` has a universal meaning.

Authentication is not enough for multi-tenant or merchant-owned data. A caller having `SCOPE_merchants.read` does not mean they may read every merchant. The service must also check the requested merchant's ownership/tenant relationship against the authenticated principal and trusted server-side data. Do not trust a caller-supplied `merchantId` as proof of access.

## 7. Use the authenticated principal safely

A controller can obtain the validated JWT principal:

```java
@GetMapping("/api/v1/me")
public Map<String, String> currentUser(@AuthenticationPrincipal Jwt jwt) {
    return Map.of("subject", jwt.getSubject());
}
```

Use the subject as the identity key only according to the identity provider's contract (often the stable combination of issuer and subject). Do not use a display name or email as a permanent authorization key unless the provider guarantees its stability and uniqueness.

The API should derive the caller identity from the validated security context, not from a request-body field such as `userId`.

## 8. 401 vs 403

| Situation | Typical response | Reason |
|---|---:|---|
| No bearer token on a protected endpoint | 401 Unauthorized | Caller is not authenticated |
| Malformed, expired, wrong-signature, wrong-issuer or wrong-audience token | 401 Unauthorized | Token cannot be accepted |
| Valid token but missing required scope | 403 Forbidden | Caller is authenticated but not authorized |
| Valid scope but caller does not own the requested merchant | 403 or a deliberate non-disclosing 404 | Object-level authorization failed |

Use a consistent error response, but do not reveal token validation internals, public-key details or sensitive account-existence information to the caller. Keep safe diagnostic details in server logs; never log the raw bearer token.

## 9. Access token, refresh token and opaque token

- The resource server should accept the intended **access token**, not a refresh token.
- The authorization server is responsible for issuing, refreshing, rotating and revoking tokens.
- A refresh token is a credential for obtaining new access tokens; it should not be sent to ordinary business APIs.
- A JWT is usually validated locally using a signature and claims. This avoids a remote introspection call on every request, but revocation may not be immediate unless the design uses short lifetimes, deny-lists or another revocation strategy.
- An opaque access token can be checked through authorization-server introspection. This can make central revocation decisions easier, but adds network latency and an availability dependency (and often needs carefully bounded caching).

Choose JWT or opaque tokens based on the system's security and operational requirements, not because one is universally better.

## 10. Test authorization without pretending to test signatures

Add `spring-security-test`. With MockMvc, Spring Security provides a `jwt()` request post-processor:

```java
@Test
void missingTokenIsRejected() throws Exception {
    mockMvc.perform(get("/api/v1/merchants/101"))
        .andExpect(status().isUnauthorized());
}

@Test
void readScopeCanAccessMerchantReadEndpoint() throws Exception {
    mockMvc.perform(get("/api/v1/merchants/101")
            .with(jwt().authorities(
                new SimpleGrantedAuthority("SCOPE_merchants.read"))))
        .andExpect(status().isOk());
}

@Test
void writeOnlyScopeCannotReadMerchant() throws Exception {
    mockMvc.perform(get("/api/v1/merchants/101")
            .with(jwt().authorities(
                new SimpleGrantedAuthority("SCOPE_merchants.write"))))
        .andExpect(status().isForbidden());
}
```

These are authorization tests. The `jwt()` helper supplies a mock authenticated JWT and bypasses the real `JwtDecoder`; it does **not** prove that a real signature, issuer, audience, expiry or JWKS key is validated.

Add a separate decoder/resource-server integration test using a local test issuer or a test JWKS endpoint and real signed test tokens. Include negative cases:

- Unknown signing key / invalid signature
- Wrong issuer
- Wrong audience
- Expired token
- Token not yet valid
- Disallowed signing algorithm
- Valid token with insufficient scope

Never use production signing keys or real user tokens in tests.

## 11. Common mistakes and trade-offs

| Mistake | Why it is dangerous | Better approach |
|---|---|---|
| Writing a custom JWT filter for standard bearer tokens | Reimplements parsing, validation and security-context behavior | Use Spring Security Resource Server |
| Checking only the signature | A correctly signed token can still target another issuer or API | Validate issuer, timestamps and audience |
| Trusting `alg` from the token header | Can create algorithm-confusion or weak-algorithm risks | Configure an explicit trusted algorithm set |
| Treating JWT payload as encrypted | Header and claims are normally readable by the token holder | Put no secrets or unnecessary personal data in claims |
| Treating every scope as an ADMIN role | Confuses delegated API permissions with application roles | Define an explicit scope/role mapping |
| Trusting a path ID after authentication | Any authenticated caller might access another tenant's object | Perform object-level authorization in the service |
| Disabling CSRF for cookie-based sessions | Browsers attach cookies automatically | Keep CSRF protection when credentials are ambient |
| Logging Authorization headers | Leaks reusable bearer credentials | Redact tokens and sensitive claims |
| Accepting refresh tokens at business endpoints | Expands the impact of token misuse | Accept only the intended access-token audience/type |
| Assuming JWT revocation is immediate | Locally validated tokens can remain valid until expiry | Choose short lifetimes or an explicit revocation design |

## 12. Practical learning exercises

- [ ] Add the Resource Server starter to a small Spring Boot MVC API.
- [ ] Configure a local/test issuer, JWKS endpoint, issuer and audience.
- [ ] Protect GET and POST merchant endpoints with different scopes.
- [ ] Add method security and a tenant/merchant ownership check.
- [ ] Write 401 and 403 MockMvc authorization tests with `jwt()`.
- [ ] Add real signed-token integration tests for signature, issuer, audience and expiry validation.
- [ ] Rotate the test JWKS key and verify that the resource server accepts the published key according to its JWKS refresh behavior.
- [ ] Document whether the API uses JWT or opaque tokens and why.
- [ ] Add a service-to-service client-credentials flow with an audience restricted to the target service.

## 13. Interview questions

1. What is the difference between OAuth 2.0, OIDC and JWT?
2. What is the difference between an authorization server and a resource server?
3. Walk through how Spring Security validates a JWT bearer token.
4. What are `iss`, `aud`, `exp`, `nbf` and `kid`?
5. Why is checking the audience as important as checking the signature?
6. What is JWKS and how does key rotation work?
7. Why does Spring map scopes to `SCOPE_` authorities?
8. Difference between `hasRole("ADMIN")` and `hasAuthority("SCOPE_admin")`?
9. Why can an authenticated user still receive 403?
10. What does `jwt()` in a MockMvc test prove, and what does it not prove?
11. When might opaque-token introspection be preferable to local JWT validation?
12. Why should a microservice not blindly forward every incoming user token to every downstream service?

## Official references

- [Spring Security — OAuth 2.0 Resource Server JWT](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/jwt.html)
- [Spring Security — Testing OAuth 2.0 / MockMvc JWT support](https://docs.spring.io/spring-security/reference/servlet/test/mockmvc/oauth2.html)
- [Spring Boot — OAuth2 Resource Server configuration](https://docs.spring.io/spring-boot/reference/security/oauth2.html)

Check the documentation for the exact Spring Boot / Spring Security version used by the sample. Let the Boot BOM manage compatible dependency versions.
