# S4.1.20 — Security Headers

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. What are Security Headers? ⭐⭐⭐⭐⭐

Security headers are HTTP response headers that instruct browsers how to handle content, framing, referrers, transport security and other browser security behavior.

```text
Client Request
      ↓
Spring Security
      ↓
Application Response
      ↓
Security Headers
      ↓
Browser applies security policy
```

Spring Security provides secure default headers for web applications, while applications can customize the policy according to their requirements.

---

# 2. Why Security Headers? ⭐⭐⭐⭐⭐

Security headers help reduce browser-side attack surface.

They can help defend against or limit impact from:

- Clickjacking
- MIME-type sniffing
- Insecure HTTP transport
- Unnecessary referrer leakage
- Unsafe browser behavior
- Some classes of XSS/content-injection impact when an appropriate CSP is configured

### Important

> **Security headers are defense-in-depth. They do not replace secure application code, input validation, output encoding, authentication or authorization.**

---

# 3. Spring Security Default Headers ⭐⭐⭐⭐⭐

Spring Security's default security-header configuration can include headers such as:

```text
Cache-Control
Content-Type
X-Content-Type-Options
Strict-Transport-Security (when applicable)
X-Frame-Options
```

The exact set and behavior depend on the Spring Security version and application configuration.

Modern Spring Security documentation should be checked for version-specific defaults.

---

# 4. `HeaderWriterFilter` ⭐⭐⭐⭐⭐

Spring Security uses `HeaderWriterFilter` to write configured security headers to the HTTP response.

Conceptually:

```text
HTTP Request
    ↓
Security Filter Chain
    ↓
HeaderWriterFilter
    ↓
Response Headers
    ↓
Browser
```

The filter delegates header generation to configured `HeaderWriter` implementations.

---

# 5. Configuring Headers ⭐⭐⭐⭐⭐

Modern Spring Security configuration uses the lambda DSL.

Example:

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http)
        throws Exception {

    http.headers(headers -> headers
        .contentTypeOptions(contentType -> {})
        .frameOptions(frame -> frame.deny())
    );

    return http.build();
}
```

Do not blindly copy every header into every application. Choose policies based on the application's architecture and browser requirements.

---

# 6. `X-Content-Type-Options` ⭐⭐⭐⭐⭐

This header helps prevent MIME-type sniffing.

Typical value:

```http
X-Content-Type-Options: nosniff
```

Conceptually:

```text
Server says:
Content-Type = application/javascript

Browser:
Do not try to reinterpret the resource as another MIME type
```

### Why important?

MIME sniffing can create security issues when browsers interpret content differently from what the server intended.

---

# 7. `X-Frame-Options` ⭐⭐⭐⭐⭐

Controls whether a page can be loaded inside a frame/iframe.

Common values:

```http
X-Frame-Options: DENY
```

or:

```http
X-Frame-Options: SAMEORIGIN
```

### DENY

The page cannot be framed.

### SAMEORIGIN

The page can be framed by pages from the same origin.

---

# 8. Clickjacking ⭐⭐⭐⭐⭐

Clickjacking tricks a user into clicking something different from what they believe they are clicking, often through an embedded page.

```text
Attacker page
    ↓
Transparent/embedded target UI
    ↓
Victim clicks
    ↓
Unintended action
```

Frame-related protections reduce this attack surface.

---

# 9. `frame-ancestors` vs `X-Frame-Options` ⭐⭐⭐⭐⭐

Modern applications should understand CSP's `frame-ancestors` directive.

```http
Content-Security-Policy: frame-ancestors 'none'
```

Conceptually:

```text
X-Frame-Options
      ↓
Older/legacy framing control

CSP frame-ancestors
      ↓
Modern CSP framing policy
```

For modern applications, `frame-ancestors` is an important policy to understand, especially when CSP is already deployed.

---

# 10. Content Security Policy (CSP) ⭐⭐⭐⭐⭐

CSP is one of the most important modern browser security policies.

It tells the browser which content sources are allowed.

Example:

```http
Content-Security-Policy: default-src 'self'
```

Conceptually:

```text
HTML page
   ↓
Browser reads CSP
   ↓
Allowed sources determined
   ↓
Unexpected scripts/resources blocked
```

---

# 11. Why CSP? ⭐⭐⭐⭐⭐

CSP can reduce the impact of certain content-injection/XSS attacks by restricting where scripts and other resources can come from.

Example:

```text
Attacker injects script
        ↓
Browser checks CSP
        ↓
Script source not allowed
        ↓
Execution can be blocked
```

### Important

> CSP is not a substitute for fixing XSS. Secure output encoding and input handling remain essential.

---

# 12. CSP Directives ⭐⭐⭐⭐⭐

Important directives include:

```text
default-src
script-src
style-src
img-src
font-src
connect-src
frame-src
frame-ancestors
object-src
base-uri
form-action
```

Example:

```http
Content-Security-Policy:
  default-src 'self';
  script-src 'self';
  style-src 'self';
  img-src 'self' https:;
  object-src 'none';
  base-uri 'self';
  frame-ancestors 'none';
```

The actual policy should be designed from the application's resource requirements.

---

# 13. `default-src` ⭐⭐⭐⭐

Acts as a fallback for resource types that do not have a more specific directive.

```http
default-src 'self'
```

Means resources should generally originate from the same origin unless a more specific directive allows another source.

---

# 14. `script-src` ⭐⭐⭐⭐⭐

Controls permitted JavaScript sources.

Example:

```http
script-src 'self'
```

Avoid overly broad policies such as:

```http
script-src * 'unsafe-inline' 'unsafe-eval'
```

unless there is a very specific, understood reason.

---

# 15. Nonce-Based CSP ⭐⭐⭐⭐⭐

For applications that require inline scripts, a nonce-based CSP is generally stronger than broadly allowing all inline scripts.

Conceptually:

```http
Content-Security-Policy: script-src 'self' 'nonce-randomValue'
```

HTML:

```html
<script nonce="randomValue">
    // trusted inline script
</script>
```

The nonce must be unpredictable and generated appropriately for the response.

---

# 16. Hash-Based CSP ⭐⭐⭐⭐

CSP can also authorize specific inline script content using hashes.

Conceptually:

```http
script-src 'self' 'sha256-...'
```

This is useful when exact script content is known and stable.

---

# 17. `unsafe-inline` and `unsafe-eval` ⭐⭐⭐⭐⭐

### `unsafe-inline`

Allows inline scripts/styles depending on directive usage.

### `unsafe-eval`

Allows JavaScript evaluation mechanisms that CSP otherwise restricts.

Broadly enabling these weakens CSP.

### Interview answer

> **Do not add `unsafe-inline` or `unsafe-eval` simply to make CSP errors disappear. First understand which resource requires them and redesign the policy/application where possible.**

---

# 18. `Strict-Transport-Security` / HSTS ⭐⭐⭐⭐⭐

Header:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

HSTS tells a browser to use HTTPS for future requests to the host for the specified period.

Conceptually:

```text
First HTTPS response
       ↓
HSTS received
       ↓
Browser remembers policy
       ↓
Future HTTP attempt
       ↓
Browser upgrades/blocks according to HSTS rules
```

---

# 19. HSTS Important Rules ⭐⭐⭐⭐⭐

HSTS should be deployed only when HTTPS is correctly configured for the relevant host/domain.

Important directives:

```text
max-age
includeSubDomains
preload
```

### `max-age`

How long the browser should remember the HTTPS policy.

### `includeSubDomains`

Extends policy to subdomains.

### `preload`

Related to browser preload lists and requires careful domain-wide HTTPS readiness.

---

# 20. HSTS and First Request Problem ⭐⭐⭐⭐

Before receiving HSTS for the first time, a browser may still attempt an HTTP connection if the user explicitly enters an HTTP URL.

HSTS preload can address this first-connection limitation for eligible domains, but it should only be used after verifying that the entire domain/subdomain deployment supports HTTPS correctly.

---

# 21. Referrer-Policy ⭐⭐⭐⭐⭐

Controls how much referrer information the browser sends when navigating to another resource/site.

Example:

```http
Referrer-Policy: strict-origin-when-cross-origin
```

This is a commonly useful privacy/security policy.

---

# 22. Why Referrer-Policy? ⭐⭐⭐⭐

URLs can sometimes contain sensitive information.

```text
https://example.com/account?some-sensitive-value
```

Without appropriate referrer controls, information about the originating URL can potentially be exposed to another destination.

Referrer-Policy helps control this behavior.

---

# 23. Cache-Control ⭐⭐⭐⭐⭐

Security-sensitive responses may require appropriate caching controls.

Typical Spring Security behavior includes headers such as:

```http
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
```

### Why?

Sensitive authenticated pages should not unnecessarily remain available from browser/proxy caches after logout or session changes.

### Important nuance

Caching should be designed per resource. Do not blindly disable caching for every public static asset.

---

# 24. `Permissions-Policy` ⭐⭐⭐⭐⭐

`Permissions-Policy` controls which browser capabilities may be used by a page and, in relevant cases, embedded content.

Example concept:

```http
Permissions-Policy: geolocation=(), camera=(), microphone=()
```

This can reduce unnecessary browser capability exposure.

### Important

The exact syntax and supported features evolve with browser standards. Verify supported directives for the target browsers.

---

# 25. `Cross-Origin-Opener-Policy` ⭐⭐⭐⭐

COOP controls the relationship between a document and other browsing contexts it opens.

Example:

```http
Cross-Origin-Opener-Policy: same-origin
```

It can help isolate a document from cross-origin opener relationships and is relevant to modern browser isolation/security architectures.

---

# 26. `Cross-Origin-Resource-Policy` ⭐⭐⭐⭐

CORP controls which origins may load a resource.

Example:

```http
Cross-Origin-Resource-Policy: same-origin
```

It can help protect resources from unwanted cross-origin inclusion.

---

# 27. `Cross-Origin-Embedder-Policy` ⭐⭐⭐⭐

COEP controls whether a document can load certain cross-origin resources unless they explicitly permit embedding.

Example:

```http
Cross-Origin-Embedder-Policy: require-corp
```

COEP is often discussed together with COOP for cross-origin isolation.

---

# 28. COOP + COEP + CORP ⭐⭐⭐⭐

These headers solve different problems.

```text
COOP
 ↓
Opener relationship

COEP
 ↓
Embedding cross-origin resources

CORP
 ↓
Who can load a resource
```

Do not treat them as interchangeable.

---

# 29. Spring Security Header DSL ⭐⭐⭐⭐⭐

Modern configuration can be composed through the headers DSL.

Example:

```java
http.headers(headers -> headers
    .frameOptions(frame -> frame.deny())
    .contentTypeOptions(contentType -> {})
    .httpStrictTransportSecurity(hsts -> hsts
        .includeSubDomains(true)
        .maxAgeInSeconds(31536000)
    )
);
```

The exact API can change between Spring Security versions, so always verify against the version used by the project.

---

# 30. Adding CSP in Spring Security ⭐⭐⭐⭐⭐

Conceptually:

```java
http.headers(headers -> headers
    .contentSecurityPolicy(csp -> csp
        .policyDirectives("default-src 'self'; object-src 'none'; frame-ancestors 'none'")
    )
);
```

### Important

Do not copy this as a universal production policy. CSP must match the application's scripts, styles, images, APIs, fonts, frames and third-party dependencies.

---

# 31. Report-Only CSP ⭐⭐⭐⭐⭐

Before enforcing a new CSP, teams can use report-only mode.

```http
Content-Security-Policy-Report-Only: default-src 'self'
```

Conceptually:

```text
Browser evaluates policy
       ↓
Violations detected
       ↓
Reports/diagnostics
       ↓
Application tunes policy
       ↓
Enforce CSP
```

This reduces the risk of breaking legitimate application resources during rollout.

---

# 32. Security Headers vs CORS ⭐⭐⭐⭐⭐

These are different concepts.

### CORS

Controls whether browser JavaScript can make/read cross-origin requests under CORS rules.

### Security Headers

Can control browser behavior such as framing, resource loading, transport and referrer handling.

```text
CORS ≠ CSP
CORS ≠ CSRF
CSP ≠ Authentication
```

---

# 33. Security Headers vs CSRF ⭐⭐⭐⭐⭐

CSRF protection prevents unauthorized state-changing requests made using a victim's authenticated context.

Security headers can reduce browser attack surface but do not replace CSRF tokens or appropriate SameSite/cookie architecture.

```text
CSRF protection
      ↓
Request authenticity

Security headers
      ↓
Browser security policy
```

---

# 34. Security Headers vs XSS ⭐⭐⭐⭐⭐

CSP can mitigate some XSS impact, but it does not make unsafe code safe.

Correct defense:

```text
Secure coding
+ Output encoding
+ Input validation where appropriate
+ Safe templating
+ CSP defense-in-depth
```

---

# 35. Removing Headers — Dangerous Practice ⭐⭐⭐⭐⭐

Avoid:

```java
http.headers(headers -> headers.disable());
```

unless there is a deliberate, documented reason.

Disabling all security headers can remove useful protections.

Prefer targeted configuration:

```text
Keep secure defaults
        ↓
Customize only what application requires
```

---

# 36. When to Customize `X-Frame-Options` ⭐⭐⭐⭐

Suppose an application must be embedded by a trusted origin.

Do not simply disable all headers.

Instead:

```text
Requirement
   ↓
Identify framing policy
   ↓
Use CSP frame-ancestors where appropriate
   ↓
Test browser behavior
```

Security policy should match the actual embedding requirement.

---

# 37. Security Headers in REST APIs ⭐⭐⭐⭐

REST APIs are not immune to security-header considerations, but the useful header set differs from a browser-rendered HTML application.

For an API:

- HTTPS/HSTS may be relevant
- MIME sniffing protection can be useful
- Caching policy matters for sensitive responses
- CORS may be important for browser clients
- CSP is mainly relevant to documents/resources interpreted by browsers, not a replacement for API authorization

### Interview point

> **Do not blindly apply an HTML security-header policy to every API response. Choose headers according to the resource and client.**

---

# 38. Security Headers with React/SPA ⭐⭐⭐⭐⭐

For a React/SPA frontend:

```text
Browser
  ↓
index.html
  ↓
CSP / security headers
  ↓
JS bundles
  ↓
API calls
```

CSP must account for:

- Script sources
- Style sources
- Fonts
- Images
- API endpoints (`connect-src`)
- Third-party analytics/payment providers
- WebSocket endpoints where used

A strict CSP is powerful but can break an SPA if dependencies are not explicitly allowed.

---

# 39. Security Headers Behind Reverse Proxy ⭐⭐⭐⭐

In production, headers may be generated by:

```text
Spring Security
      OR
Reverse Proxy / Gateway
      OR
CDN
      OR
Web Server
```

Avoid accidental conflicting policies.

Example:

```text
Browser
  ↓
CDN
  ↓
Gateway
  ↓
Spring Boot
```

Define ownership clearly and verify the final headers received by the browser.

---

# 40. Testing Security Headers ⭐⭐⭐⭐⭐

Use browser developer tools or HTTP clients.

Example:

```bash
curl -I https://example.com
```

Check for headers such as:

```text
Strict-Transport-Security
Content-Security-Policy
X-Content-Type-Options
X-Frame-Options
Referrer-Policy
Cache-Control
Permissions-Policy
```

Also test actual behavior, not only header presence.

---

# 41. Production Checklist ⭐⭐⭐⭐⭐

```text
[ ] HTTPS everywhere
[ ] HSTS correctly configured
[ ] X-Content-Type-Options appropriate
[ ] Frame protection configured
[ ] CSP designed and tested
[ ] Referrer-Policy reviewed
[ ] Cache policy reviewed
[ ] Permissions-Policy reviewed
[ ] CORS explicitly configured where required
[ ] CSRF architecture reviewed for cookie authentication
[ ] Secure/HttpOnly/SameSite cookies
[ ] Headers not duplicated/conflicting at proxy layer
[ ] CSP report-only rollout considered
[ ] Sensitive responses not unnecessarily cached
```

---

# 42. Common Interview Traps ⭐⭐⭐⭐⭐

### Trap 1 — CSP prevents all XSS

❌ Wrong.

> CSP is defense-in-depth; application-level XSS prevention remains necessary.

### Trap 2 — `X-Frame-Options` and CSP are identical

❌ Wrong.

> CSP `frame-ancestors` is the modern framing policy to understand; X-Frame-Options remains relevant for compatibility.

### Trap 3 — CORS is a security header that prevents CSRF

❌ Wrong.

> CORS and CSRF address different browser security problems.

### Trap 4 — HSTS encrypts traffic

❌ Wrong.

> HTTPS provides encryption; HSTS tells browsers to use HTTPS for the host according to the policy.

### Trap 5 — `X-XSS-Protection` should always be enabled

❌ Outdated.

> Modern browsers have deprecated/removed reliance on the old XSS auditor behavior. Prefer CSP and secure coding rather than depending on `X-XSS-Protection`.

### Trap 6 — Disable all headers if one header causes an issue

❌ Bad practice.

> Customize the specific header/policy instead.

### Trap 7 — Security headers are only for frontend developers

❌ Wrong.

> Backend, gateway and security configuration all contribute to the final HTTP response policy.

---

# 43. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What are security headers?

> HTTP response headers that instruct browsers to apply security-related behavior, such as HTTPS enforcement, framing restrictions, MIME handling and content-loading policies.

### Q2. What is `X-Content-Type-Options: nosniff`?

> It tells browsers not to MIME-sniff a response away from the declared content type.

### Q3. What is `X-Frame-Options`?

> It controls whether a page can be loaded inside a frame and helps mitigate clickjacking.

### Q4. What is CSP?

> Content Security Policy is a browser-enforced policy controlling permitted content sources and can reduce the impact of some content-injection/XSS attacks.

### Q5. What is HSTS?

> HTTP Strict Transport Security tells browsers to use HTTPS for future requests to a host for a configured period.

### Q6. CSP vs CORS?

> CSP controls browser resource-loading/content policies; CORS controls cross-origin browser request access under CORS rules.

### Q7. Does Spring Security automatically configure CSP?

> Spring Security provides APIs for CSP configuration, but applications generally need to define an appropriate CSP policy for their resource requirements rather than assuming a universal CSP.

### Q8. Why not disable all security headers?

> Because secure defaults provide defense-in-depth. Only the specific policy that conflicts with a legitimate application requirement should be customized.

### Q9. Why use `Content-Security-Policy-Report-Only`?

> To observe policy violations and tune a CSP before enforcing it.

### Q10. Does HSTS replace HTTPS?

> No. HTTPS provides secure transport; HSTS tells browsers to consistently use HTTPS after the policy is learned.

---

# 44. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“Security headers are HTTP response headers that instruct browsers to apply additional security policies. Spring Security provides secure defaults and a header DSL for customization. Important headers include X-Content-Type-Options for preventing MIME sniffing, X-Frame-Options and CSP frame-ancestors for clickjacking protection, HSTS for enforcing HTTPS after the browser learns the policy, Referrer-Policy for controlling referrer information, and Cache-Control for sensitive response caching behavior. CSP is especially important because it restricts allowed content sources and can reduce the impact of some XSS attacks, although it does not replace secure coding. In production, I would design CSP based on actual application dependencies, potentially roll it out with report-only mode first, keep HTTPS enabled, protect cookies, and verify the final headers at the browser because headers can also be added or modified by a gateway, CDN or reverse proxy. I would also distinguish security headers from CORS and CSRF because they solve different problems.”**

---

# 45. 30-Second Hinglish Answer

> **“Security headers browser ko security-related instructions dete hain. Spring Security secure defaults aur headers DSL provide karta hai. Important headers hain `X-Content-Type-Options` for MIME sniffing protection, `X-Frame-Options`/CSP `frame-ancestors` for clickjacking, HSTS for HTTPS enforcement, Referrer-Policy for referrer control aur CSP for resource/script restrictions. CSP XSS ko completely solve nahi karta, but defense-in-depth deta hai. CORS aur CSRF bhi security headers se different concepts hain. Production mein headers ko application ke actual requirements ke according configure aur browser level par verify karna chahiye.”**

---

# 46. Whiteboard Memory ⭐⭐⭐⭐⭐

```text
                  HTTP RESPONSE
                        ↓
             ┌──────────┼──────────┐
             ↓          ↓          ↓
            HSTS       CSP      Frame Policy
             ↓          ↓          ↓
          HTTPS      Resources   Clickjacking

             ↓          ↓          ↓
         Referrer    MIME       Cache
          Policy     nosniff     Control
             ↓          ↓          ↓
              Browser Security
```

### Final memory line

> **Security Headers = Browser Defense-in-Depth → HSTS (HTTPS) + CSP (resources/XSS defense) + Frame Protection (clickjacking) + nosniff (MIME) + Referrer/Cache/Permissions policies.**

---

## Navigation

[← S4.1.19 — Remember-Me Authentication](../19-Remember-Me-Authentication/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.21 — OAuth2 Fundamentals**
