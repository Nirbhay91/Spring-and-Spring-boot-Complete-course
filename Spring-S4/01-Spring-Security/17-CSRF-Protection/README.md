# S4.1.17 — CSRF Protection

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. What is CSRF? ⭐⭐⭐⭐⭐

**CSRF = Cross-Site Request Forgery**.

CSRF ek attack hai jisme attacker victim ke browser se kisi trusted application par unwanted state-changing request karwane ki koshish karta hai.

Typical condition:

```text
Victim browser
   ↓
Already authenticated
   ↓
Has session cookie
   ↓
Attacker tricks browser into sending request
   ↓
Target application receives trusted cookie
   ↓
Unwanted action
```

### Golden rule

> **CSRF mainly browser-based applications mein dangerous hota hai jab browser automatically authentication credentials, especially cookies, request ke saath send karta hai.**

---

# 2. Simple Real-World Example ⭐⭐⭐⭐⭐

Suppose bank application:

```text
POST /transfer
Cookie: SESSION=abc123
```

User bank mein already logged in hai.

Attacker ek malicious page banata hai jo victim ke browser se transfer request trigger karne ki koshish karta hai.

```text
Attacker page
      ↓
Victim browser
      ↓
POST /transfer
      ↓
Browser automatically sends bank session cookie
      ↓
Bank sees authenticated session
```

Without CSRF protection, application may incorrectly trust the request.

---

# 3. Why Does CSRF Work? ⭐⭐⭐⭐⭐

Core reason:

```text
Browser
  ↓
automatically attaches cookie
  ↓
Server sees valid session
```

Server can identify the user but may not know whether the request was intentionally initiated by the user.

CSRF protection adds an additional proof that the request originated from the legitimate application flow.

---

# 4. CSRF vs Authentication ⭐⭐⭐⭐⭐

CSRF does **not** primarily mean the attacker stole the user's password.

Instead:

```text
User is already authenticated
        ↓
Attacker abuses browser's authenticated context
        ↓
Unwanted request
```

### Memory

> **Authentication says who the user is; CSRF protection helps verify that a state-changing browser request was intentionally initiated by the application/user flow.**

---

# 5. CSRF vs XSS ⭐⭐⭐⭐⭐

These are different attacks.

### CSRF

Attacker tries to make the victim's authenticated browser send an unwanted request.

```text
Attacker → tricks browser → target application
```

### XSS

Attacker injects executable script/content into a trusted page/context.

```text
Attacker input
   ↓
Application output
   ↓
Victim browser executes malicious script
```

### Important

> Strong CSRF protection does not replace XSS protection, and XSS can undermine many CSRF defenses because malicious script may be able to access tokens available to the page.

---

# 6. CSRF Token ⭐⭐⭐⭐⭐

A common defense is a CSRF token.

Conceptually:

```text
Server generates token
       ↓
Legitimate page receives token
       ↓
Client sends token with state-changing request
       ↓
Server validates token
```

Example:

```text
Cookie:
SESSION=abc123

Request body/header:
CSRF_TOKEN=xyz789
```

The attacker may cause a browser to send the session cookie, but should not be able to obtain the legitimate CSRF token in a correctly designed same-origin application.

---

# 7. Synchronizer Token Pattern ⭐⭐⭐⭐⭐

Traditional session-based applications often use the synchronizer token pattern.

```text
Server session
   ├── Authentication state
   └── CSRF token

Legitimate form
   ↓
Contains CSRF token
   ↓
POST request
   ↓
Server compares expected vs supplied token
```

If token is missing or invalid:

```text
Request rejected
```

---

# 8. SameSite Cookies ⭐⭐⭐⭐⭐

Modern browsers support cookie `SameSite` behavior that can reduce CSRF risk.

Common values:

```text
Strict
Lax
None
```

Conceptually:

- `Strict` — strongest cross-site restriction.
- `Lax` — allows some cross-site cookie sending in limited navigation scenarios.
- `None` — cookie can be sent cross-site and generally requires `Secure`.

### Important

> SameSite is an important browser-level mitigation, but application security should still be designed according to the authentication mechanism and browser/client behavior.

---

# 9. Spring Security CSRF Default ⭐⭐⭐⭐⭐

For applications where CSRF protection is enabled, Spring Security provides CSRF protection through its security infrastructure.

Typical configuration:

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http)
        throws Exception {

    http
        .csrf(csrf -> csrf
            .csrfTokenRepository(csrfTokenRepository())
        );

    return http.build();
}
```

The exact defaults and behavior depend on the application type and Spring Security version, so current configuration should be checked against the version used by the project.

---

# 10. Why Does Spring Security Protect Against CSRF? ⭐⭐⭐⭐⭐

Because many Spring applications historically use browser-based authentication with cookies/sessions.

Example:

```text
Browser
  ↓
SESSION cookie
  ↓
Spring Security
  ↓
Authenticated request
```

A malicious cross-site request can potentially reuse that browser authentication context.

CSRF protection helps prevent unauthorized state-changing requests.

---

# 11. Which HTTP Methods Matter? ⭐⭐⭐⭐⭐

CSRF is primarily a concern for **state-changing operations**.

Typical examples:

```text
POST
PUT
PATCH
DELETE
```

Safe/read-only operations are generally expected to be:

```text
GET
HEAD
OPTIONS
```

### Important interview point

> **Do not perform state-changing operations through GET.**

Bad:

```text
GET /deleteUser?id=10
```

Better:

```text
DELETE /users/10
```

---

# 12. CSRF Token in HTML Form ⭐⭐⭐⭐⭐

Server-side rendered form can contain a hidden CSRF token.

Conceptually:

```html
<form method="post" action="/transfer">
    <input type="hidden"
           name="_csrf"
           value="..." />

    <button type="submit">Transfer</button>
</form>
```

The exact token rendering depends on the template technology and Spring Security integration.

---

# 13. CSRF Token in AJAX / Fetch ⭐⭐⭐⭐⭐

For JavaScript-based browser clients, a CSRF token can be sent using a request header.

Conceptually:

```javascript
fetch('/api/orders', {
    method: 'POST',
    headers: {
        'X-CSRF-TOKEN': csrfToken,
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(order)
});
```

The server validates the token before processing the state-changing request.

---

# 14. `CookieCsrfTokenRepository` ⭐⭐⭐⭐⭐

For JavaScript/browser applications, Spring Security provides `CookieCsrfTokenRepository`.

Example:

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http)
        throws Exception {

    http.csrf(csrf -> csrf
        .csrfTokenRepository(
            CookieCsrfTokenRepository.withHttpOnlyFalse()
        )
    );

    return http.build();
}
```

Why `withHttpOnlyFalse()`?

```text
JavaScript needs to read the CSRF token
```

So the CSRF cookie may need to be readable by client-side JavaScript.

### Important security nuance

> This does **not** mean the authentication/session cookie should be made JavaScript-readable. The CSRF token and authentication credential have different security properties.

---

# 15. CSRF Header Names ⭐⭐⭐⭐

A common setup uses:

```text
Cookie: XSRF-TOKEN=...
Header: X-XSRF-TOKEN: ...
```

Spring Security can be configured to use appropriate cookie/header conventions.

Do not hard-code a frontend/backend contract without checking the actual repository configuration.

---

# 16. CSRF and Stateless JWT APIs ⭐⭐⭐⭐⭐

This is one of the most important interview topics.

Many stateless APIs use:

```text
Authorization: Bearer <JWT>
```

The browser does not automatically attach an Authorization bearer token to arbitrary cross-site requests in the same way it automatically attaches cookies.

Therefore, a typical stateless bearer-token API may not need traditional CSRF protection **for that authentication mechanism**.

But:

> **Stateless does not automatically mean CSRF-safe. The real question is whether the browser automatically sends the authentication credential cross-site.**

---

# 17. Cookie-Based JWT vs Authorization Header ⭐⭐⭐⭐⭐

### JWT in Authorization header

```text
Authorization: Bearer <JWT>
```

JavaScript/client explicitly supplies the token.

Traditional CSRF risk is generally reduced because the browser does not automatically attach that header cross-site.

### JWT in cookie

```text
Cookie: accessToken=...
```

Browser automatically attaches the cookie according to cookie rules.

CSRF protection may still be required.

### Golden rule

> **CSRF is about automatic credential transmission, not about whether the credential happens to be a JWT.**

---

# 18. When NOT to Disable CSRF ⭐⭐⭐⭐⭐

Do not blindly write:

```java
.csrf(csrf -> csrf.disable())
```

for every application.

Avoid disabling CSRF in browser/session applications merely because the application is a REST API.

First determine:

```text
How is authentication stored?
        ↓
Cookie/session?
Bearer header?
Other browser credential?
        ↓
Can browser automatically attach it cross-site?
```

---

# 19. When Disabling CSRF Can Be Appropriate ⭐⭐⭐⭐⭐

A common case is a stateless API where authentication is supplied through a bearer token in the `Authorization` header and the application is not relying on browser cookies for authentication.

Example concept:

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http)
        throws Exception {

    http
        .csrf(csrf -> csrf.disable())
        .authorizeHttpRequests(auth -> auth
            .anyRequest().authenticated()
        )
        .oauth2ResourceServer(oauth2 -> oauth2.jwt());

    return http.build();
}
```

### Important

This is an architectural decision, not a universal rule.

---

# 20. CSRF and REST APIs ⭐⭐⭐⭐⭐

REST itself does **not** automatically mean CSRF is irrelevant.

Wrong assumption:

```text
REST API = CSRF not needed
```

Correct reasoning:

```text
Authentication mechanism
        ↓
Does browser automatically send credential?
        ↓
CSRF exposure
        ↓
Choose appropriate protection
```

---

# 21. CSRF and CORS ⭐⭐⭐⭐⭐

CSRF and CORS solve different problems.

### CORS

Controls whether browser JavaScript from one origin can make/read cross-origin requests under browser CORS rules.

### CSRF

Protects against unwanted state-changing requests made using the victim's authenticated browser context.

### Memory

```text
CORS → cross-origin browser access policy
CSRF → forged state-changing request defense
```

### Important

> **CORS is not a replacement for CSRF protection.**

---

# 22. CSRF and `Origin` / `Referer` ⭐⭐⭐⭐

Applications can also use request-origin information as part of CSRF defenses.

Potential headers:

```text
Origin
Referer
```

But they should be treated as part of a deliberate security design rather than as a casual replacement for Spring Security's CSRF token mechanism.

---

# 23. CSRF and Same-Origin Policy ⭐⭐⭐⭐⭐

The browser's Same-Origin Policy helps prevent an attacker origin from freely reading protected content from another origin.

But it does not necessarily prevent the attacker from **sending** every kind of cross-origin request.

That's why CSRF remains relevant.

```text
Same-Origin Policy
       ↓
Limits cross-origin reads
       ≠
Automatically prevents every cross-origin write
```

---

# 24. CSRF Token Validation Flow ⭐⭐⭐⭐⭐

```text
Browser
   ↓
GET legitimate page
   ↓
CSRF token issued/exposed
   ↓
User submits form/API request
   ↓
CSRF token sent
   ↓
Spring Security CSRF validation
   ↓
 ┌───────────────────┐
 │ Token valid?      │
 └─────────┬─────────┘
       YES │ NO
           │
      ↓    │    ↓
  Continue │ Reject
  request  │
           ↓
    CSRF failure
```

---

# 25. Invalid CSRF Token ⭐⭐⭐⭐⭐

If a protected state-changing request does not contain the expected CSRF token, Spring Security can reject it.

Conceptually:

```text
POST /orders
       ↓
No valid CSRF token
       ↓
CSRF validation fails
       ↓
Request rejected
```

This failure should be handled consistently with the application's error-response strategy.

---

# 26. `CsrfTokenRequestHandler` ⭐⭐⭐⭐

Modern Spring Security versions provide `CsrfTokenRequestHandler` as part of the CSRF token request-handling infrastructure.

It can influence how the CSRF token is resolved/exposed for requests.

This becomes especially relevant for JavaScript clients and newer Spring Security configurations.

---

# 27. CSRF Token Repository ⭐⭐⭐⭐⭐

A `CsrfTokenRepository` abstracts how CSRF tokens are generated and stored.

Conceptually:

```text
CsrfTokenRepository
       ↓
Generate / save / load token
       ↓
CSRF validation
```

Common implementation:

```text
CookieCsrfTokenRepository
```

Other application designs may use a session-backed token repository.

---

# 28. Session-Based CSRF Token ⭐⭐⭐⭐

For server-side applications, the CSRF token can be associated with the server-side session.

```text
HTTP Session
   ├── Authentication
   └── CSRF token
```

Request:

```text
Session cookie + CSRF token
```

Server verifies both pieces of information.

---

# 29. CSRF and Login ⭐⭐⭐⭐

CSRF protection is not only about money transfers or deletes.

Authentication-related state changes can also be security-sensitive.

Examples:

```text
Change email
Change password
Enable MFA
Link account
Change payment details
```

The application should protect relevant state-changing browser requests appropriately.

---

# 30. CSRF and Logout ⭐⭐⭐⭐

Logout is also a state-changing operation.

If logout is implemented through a state-changing request, CSRF considerations apply.

Avoid designing security-sensitive state changes as unsafe GET endpoints merely to bypass CSRF concerns.

---

# 31. Testing CSRF ⭐⭐⭐⭐⭐

Test:

```text
POST without CSRF token → rejected
POST with invalid token → rejected
POST with valid token → allowed
GET/read-only request → behaves as configured
```

For REST APIs, also test the exact authentication mechanism:

```text
Bearer token
Cookie session
JWT cookie
Browser client
Non-browser client
```

---

# 32. CSRF in Microservices ⭐⭐⭐⭐⭐

For microservices, first identify the client-to-service authentication mechanism.

### Browser session architecture

```text
Browser
  ↓
Cookie
  ↓
Gateway/service
  ↓
CSRF protection
```

### Bearer-token API architecture

```text
Browser / Client
  ↓
Authorization: Bearer JWT
  ↓
API Gateway
  ↓
Microservice
```

Traditional CSRF requirements can differ significantly between these architectures.

---

# 33. Production Design ⭐⭐⭐⭐⭐

### Browser + Session/Cookie

```text
Authentication: Cookie/Session
        ↓
CSRF protection enabled
        ↓
CSRF token + SameSite cookie strategy
        ↓
State-changing request protected
```

### Stateless Bearer API

```text
Authentication: Authorization Bearer JWT
        ↓
No browser-authentication cookie dependency
        ↓
Traditional CSRF protection may be disabled
        ↓
Still enforce authentication + authorization
```

---

# 34. Common Mistakes ⭐⭐⭐⭐⭐

### Mistake 1

> “CSRF is only a frontend problem.”

❌ No. It is a server-side security policy involving browser behavior.

### Mistake 2

> “REST API means disable CSRF.”

❌ Not automatically.

### Mistake 3

> “JWT means CSRF is impossible.”

❌ Depends on where/how JWT is transported.

### Mistake 4

> “CORS protects against CSRF.”

❌ Different security mechanisms.

### Mistake 5

> “SameSite means CSRF is solved forever.”

❌ Defense-in-depth and browser/client behavior still matter.

### Mistake 6

> “Just use GET for delete.”

❌ Unsafe and can create serious security problems.

### Mistake 7

> “Disable CSRF in every Spring Boot REST application.”

❌ Architecture must be evaluated first.

---

# 35. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is CSRF?

> CSRF is an attack where an attacker causes a victim's authenticated browser to submit an unwanted state-changing request to a trusted application.

### Q2. Why is CSRF mainly associated with cookies?

> Browsers automatically attach cookies to matching requests, so an attacker may be able to trigger a request that carries the victim's authenticated cookie.

### Q3. How does CSRF token protect the application?

> The server expects an unpredictable token in addition to the authentication context. A cross-site attacker should not be able to obtain and correctly submit that token.

### Q4. CSRF vs CORS?

> CORS controls browser cross-origin access rules; CSRF protects against forged state-changing requests. CORS is not a replacement for CSRF protection.

### Q5. Should CSRF be disabled for JWT APIs?

> Not simply because they use JWT. If the JWT is sent in an Authorization bearer header and the browser does not automatically attach the credential cross-site, traditional CSRF risk is usually reduced. If the JWT is stored in a cookie, CSRF may still matter.

### Q6. Why is GET generally not protected like POST?

> GET should be safe/read-only. State-changing actions should use appropriate methods such as POST, PUT, PATCH or DELETE and should be protected against CSRF where applicable.

### Q7. What is `CookieCsrfTokenRepository`?

> It is a Spring Security repository that stores/exposes CSRF token information using a cookie-based mechanism, commonly useful for JavaScript browser clients.

### Q8. What is `withHttpOnlyFalse()`?

> It configures the CSRF cookie so browser JavaScript can read the token. This should not be confused with making the authentication cookie JavaScript-readable.

### Q9. Does SameSite replace CSRF tokens?

> SameSite provides an important browser-level mitigation, but whether it is sufficient depends on the application's architecture, browser behavior and threat model. Defense in depth is preferred.

### Q10. What is the main question before disabling CSRF?

> How is authentication transported, and can the browser automatically send that credential in a cross-site request?

---

# 36. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“CSRF stands for Cross-Site Request Forgery. It occurs when an attacker causes a victim's already-authenticated browser to send an unwanted state-changing request to a trusted application. It is particularly relevant to cookie or session-based authentication because browsers automatically attach those credentials. Spring Security can protect state-changing requests using CSRF tokens, where the legitimate client sends a token that the attacker should not be able to obtain. SameSite cookies provide another browser-level mitigation, but CORS and SameSite should not be treated as universal replacements for CSRF protection. For stateless APIs using bearer tokens in the Authorization header, traditional CSRF risk is generally reduced because the browser does not automatically attach that Authorization header cross-site; therefore many such APIs disable CSRF. But if a JWT is stored in a cookie, CSRF can still be relevant. The key architectural question is not ‘session or JWT’, but whether the browser automatically sends the authentication credential cross-site.”**

---

# 37. 30-Second Hinglish Answer

> **“CSRF ek attack hai jisme attacker victim ke already-authenticated browser se unwanted state-changing request karwata hai. Ye mainly cookie/session authentication mein important hai because browser automatically cookie bhej deta hai. Spring Security CSRF token ke through request ko validate kar sakta hai. `SameSite` cookies bhi mitigation provide karti hain, lekin CORS CSRF ka replacement nahi hai. Stateless JWT API mein agar JWT `Authorization: Bearer` header se explicitly send hota hai, traditional CSRF risk generally low hota hai, isliye CSRF disable kiya ja sakta hai. Lekin JWT cookie mein stored hai to CSRF still relevant ho sakta hai.”**

---

# 38. Whiteboard Memory ⭐⭐⭐⭐⭐

```text
                 BROWSER
                    ↓
          Authentication Cookie
                    ↓
          Cross-Site Request
                    ↓
               TARGET API
                    ↓
              CSRF Check
                    ↓
          ┌─────────┴─────────┐
          ↓                   ↓
       VALID                 INVALID
          ↓                   ↓
       ALLOW                REJECT
```

### Final memory line

> **CSRF = attacker abuses automatically attached browser credentials → protect state-changing requests → token/SameSite/appropriate architecture → don't blindly disable CSRF.**

---

## Navigation

[← S4.1.16 — Exception Handling in Spring Security](../16-Exception-Handling-in-Spring-Security/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.18 — Session Management**
