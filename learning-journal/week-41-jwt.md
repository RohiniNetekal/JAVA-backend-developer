# One-Page Concept Reference

**Status:** ✅ Done — reference example created

## JWT Basics

### Short explanation
A JWT contains encoded claims and a signature. The server validates the token before trusting its claims.

### Handy example
```text
header.payload.signature
```

### Key point
Remember: JWT is signed, not automatically encrypted; do not put secrets in ordinary claims.

### Interview check
- What is it?
- Why would you use it?
- What is one common mistake or trade-off?


## What to cover
- [ ] Header/payload/signature
- [ ] Claims
- [ ] Expiration
- [ ] Issuer/audience
- [ ] Signature validation
- [ ] Access vs refresh tokens
- [ ] Stateless authentication
- [ ] Do not store secrets in claims

## Concept in slightly more detail
JWT carries signed claims that the server validates before using. Signing does not mean encryption.

## Practical example
```text
header.payload.signature
```

## Done checklist
- [x] Reference created
- [ ] All checklist points understood
- [ ] Example explained without notes
- [ ] 3 interview questions practiced

## Practical extension — validating a JWT access token

A resource server must not trust claims merely because the token can be Base64-decoded. Verify the signature using a trusted key (commonly obtained from the issuer's JWKS endpoint), then validate at least:

- iss: expected issuer
- aud: this API is an intended recipient
- exp and nbf: token time validity
- allowed signature algorithm and key

The kid header helps select a key from JWKS and supports key rotation. A JWT is signed, not automatically encrypted.

An API should accept the intended access token, not a refresh token. JWT validation is often local, so revocation may not be immediate; opaque-token introspection offers a different latency/availability trade-off.

For implementation, negative validation tests and the distinction between MockMvc's mock jwt() helper and real signature validation, see the [OAuth2 JWT Resource Server guide](https://github.com/RohiniNetekal/JAVA-backend-developer/blob/main/spring-security/jwt/OAUTH2_RESOURCE_SERVER_IMPLEMENTATION.md).
