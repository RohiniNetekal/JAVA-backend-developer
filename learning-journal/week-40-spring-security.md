# One-Page Concept Reference

**Status:** ✅ Done — reference example created

## Spring Security Basics

### Short explanation
Authentication answers who the caller is; authorization answers what that caller can access.

### Handy example
```java
@PreAuthorize("hasRole('ADMIN')")
public void approveOffer(Long id) { }
```

### Key point
Remember: Secure endpoints at the appropriate boundary and validate permissions server-side.

### Interview check
- What is it?
- Why would you use it?
- What is one common mistake or trade-off?


## What to cover
- [ ] Authentication
- [ ] Authorization
- [ ] Security filter chain
- [ ] Password encoding
- [ ] Roles/authorities
- [ ] Method security
- [ ] CSRF concept
- [ ] CORS concept

## Concept in slightly more detail
Authentication identifies the caller; authorization checks permissions. Security must be enforced server-side.

## Practical example
```text
@PreAuthorize("hasRole('ADMIN')") public void approveOffer(Long id) {}
```

## Done checklist
- [x] Reference created
- [ ] All checklist points understood
- [ ] Example explained without notes
- [ ] 3 interview questions practiced

## Practical extension — SecurityFilterChain and authorization

For a stateless API that receives bearer tokens in the HTTP Authorization header, Spring Security can configure an OAuth2 Resource Server through a SecurityFilterChain. Do not write a custom JWT filter for the standard bearer-token flow.

Example authorization rules:

~~~java
.requestMatchers(HttpMethod.GET, "/api/v1/merchants/**")
    .hasAuthority("SCOPE_merchants.read")
.requestMatchers(HttpMethod.POST, "/api/v1/merchants/**")
    .hasAuthority("SCOPE_merchants.write")
.anyRequest().authenticated()
~~~

Spring maps OAuth scopes to SCOPE_ authorities by default. This is not the same as an application role such as ROLE_ADMIN.

- 401: the caller is missing or has an invalid token.
- 403: the token is valid but lacks permission.
- A valid scope still does not prove ownership of a particular merchant; check tenant/object ownership in the service.

Read the [OAuth2 JWT Resource Server implementation guide](https://github.com/RohiniNetekal/JAVA-backend-developer/blob/main/spring-security/jwt/OAUTH2_RESOURCE_SERVER_IMPLEMENTATION.md) for the complete filter chain, CSRF/CORS distinction, method security, object-level authorization, testing and common mistakes.
