# S4.1.11 — UsernamePasswordAuthenticationToken

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. Big Picture ⭐⭐⭐⭐⭐

`UsernamePasswordAuthenticationToken` Spring Security ka `Authentication` implementation hai jo commonly username/password authentication request aur successful authenticated principal ko represent karta hai.

```text
username + password
        ↓
UsernamePasswordAuthenticationToken
        ↓
AuthenticationManager
        ↓
ProviderManager
        ↓
DaoAuthenticationProvider
        ↓
Authenticated Authentication
```

### Memory trick

```text
UsernamePasswordAuthenticationToken =
username/password ko Authentication object ke form mein carry karna
```

---

# 2. What is UsernamePasswordAuthenticationToken?

Ye `Authentication` interface ka implementation hai.

Simplified hierarchy:

```text
Authentication
      ↑
AbstractAuthenticationToken
      ↑
UsernamePasswordAuthenticationToken
```

Iska common use username/password authentication ke liye hota hai.

---

# 3. Why do we need an Authentication Token? ⭐⭐⭐⭐⭐

Spring Security ko credentials ko directly method parameters mein pass karne ke bajay ek standard `Authentication` abstraction chahiye.

```text
username
password
   ↓
Authentication Token
   ↓
AuthenticationManager
```

Token authentication request ko security infrastructure ke andar standard format mein represent karta hai.

---

# 4. Two Important States ⭐⭐⭐⭐⭐

`UsernamePasswordAuthenticationToken` ko do conceptual states mein samjho:

### Before authentication

```text
Principal   = username
Credentials = password
Authenticated = false
```

### After successful authentication

```text
Principal   = authenticated user
Credentials = usually no longer needed
Authorities = roles/permissions
Authenticated = true
```

Important:

> **Authentication token ka object same conceptual authentication flow mein request ko represent kar sakta hai, lekin successful authentication ke liye provider normally appropriate authenticated token return karta hai.**

---

# 5. Unauthenticated Constructor ⭐⭐⭐⭐⭐

Common constructor:

```java
new UsernamePasswordAuthenticationToken(
    username,
    password
);
```

Conceptually:

```text
Principal   = username
Credentials = password
Authorities = empty
Authenticated = false
```

Example:

```java
Authentication authentication =
    new UsernamePasswordAuthenticationToken(
        request.username(),
        request.password()
    );
```

Then:

```java
Authentication result =
    authenticationManager.authenticate(authentication);
```

---

# 6. Authenticated Constructor ⭐⭐⭐⭐⭐

There is also a constructor/factory path for an already authenticated representation with authorities.

Conceptually:

```java
new UsernamePasswordAuthenticationToken(
    principal,
    credentials,
    authorities
);
```

Example:

```java
UsernamePasswordAuthenticationToken authenticated =
    new UsernamePasswordAuthenticationToken(
        userDetails,
        null,
        userDetails.getAuthorities()
    );
```

### Important security point

Application code ko user-supplied data se directly an authenticated token create karke trust nahi karna chahiye. Authenticated state should come from trusted authentication logic.

---

# 7. Factory Methods ⭐⭐⭐⭐⭐

Modern Spring Security code mein authenticated/unauthenticated intent ko explicit rakhne ke liye factory methods useful hain.

```java
UsernamePasswordAuthenticationToken.unauthenticated(
    username,
    password
);
```

And for trusted authenticated state:

```java
UsernamePasswordAuthenticationToken.authenticated(
    principal,
    credentials,
    authorities
);
```

### Interview tip

Factories ka benefit:

> **Code se authentication state ka intent clearer ho jata hai.**

---

# 8. `getPrincipal()`

Principal identity ko represent karta hai.

Before authentication:

```java
Authentication auth =
    new UsernamePasswordAuthenticationToken(
        "nirbhay",
        "secret"
    );

Object principal = auth.getPrincipal();
```

Conceptually:

```text
principal = "nirbhay"
```

After successful authentication, principal commonly `UserDetails` ya configured principal representation ho sakta hai.

---

# 9. `getCredentials()`

Credentials authentication proof ko represent karte hain.

For username/password:

```text
credentials = password
```

Example:

```java
Object credentials = authentication.getCredentials();
```

### Security rule ⭐⭐⭐⭐⭐

Password ko logs mein print mat karo:

```java
// ❌ Never do this
log.info("password={}", authentication.getCredentials());
```

Successful authentication ke baad credentials generally retain karne ki zarurat nahi hoti. Spring Security credential-erasure mechanisms support karta hai.

---

# 10. `getAuthorities()`

Authorities authenticated user ke permissions/roles represent karti hain.

```java
Collection<? extends GrantedAuthority> authorities =
    authentication.getAuthorities();
```

Example:

```text
ROLE_USER
ORDER_READ
ORDER_WRITE
```

Relationship:

```text
UsernamePasswordAuthenticationToken
          ↓
Authentication
          ↓
Authorities
          ↓
Authorization
```

---

# 11. `isAuthenticated()` ⭐⭐⭐⭐⭐

Authentication object authenticated state expose karta hai:

```java
boolean authenticated =
    authentication.isAuthenticated();
```

But important:

> **`isAuthenticated()` ko blindly trust boundary/security check ke replacement ke roop mein use nahi karna chahiye.**

Authorization should be performed through Spring Security's authorization mechanisms.

---

# 12. Complete Username/Password Flow ⭐⭐⭐⭐⭐

```text
HTTP Login Request
      ↓
username + password
      ↓
UsernamePasswordAuthenticationToken
      ↓
AuthenticationManager
      ↓
ProviderManager
      ↓
DaoAuthenticationProvider
      ↓
UserDetailsService
      ↓
UserDetails
      ↓
PasswordEncoder
      ↓
Authenticated Authentication
      ↓
SecurityContext
```

Ye S4.1.9 aur S4.1.10 se direct connection hai.

---

# 13. REST Login Example ⭐⭐⭐⭐⭐

Example:

```java
@PostMapping("/login")
public TokenResponse login(@RequestBody LoginRequest request) {

    Authentication authentication =
        authenticationManager.authenticate(
            UsernamePasswordAuthenticationToken.unauthenticated(
                request.username(),
                request.password()
            )
        );

    // authentication successful
    // generate application token if using custom login/JWT architecture

    return tokenService.createToken(authentication);
}
```

### Flow

```text
LoginRequest
    ↓
Token creation
    ↓
AuthenticationManager
    ↓
Provider
    ↓
Validation
    ↓
Authenticated Authentication
```

---

# 14. Why not directly call UserDetailsService?

Wrong architecture:

```text
Controller
   ↓
UserDetailsService
   ↓
Password comparison manually
```

Better:

```text
Controller / Filter
      ↓
AuthenticationManager
      ↓
AuthenticationProvider
      ↓
UserDetailsService + PasswordEncoder
```

Authentication concerns ko Spring Security infrastructure ke through handle karna cleaner aur safer hota hai.

---

# 15. Relationship with AuthenticationManager ⭐⭐⭐⭐⭐

```text
UsernamePasswordAuthenticationToken
             ↓
      AuthenticationManager
             ↓
       ProviderManager
             ↓
    AuthenticationProvider
```

Token **authentication request ko represent karta hai**; manager us request ko authenticate karne ke liye providers ko delegate karta hai.

---

# 16. Relationship with AuthenticationProvider ⭐⭐⭐⭐⭐

`DaoAuthenticationProvider` commonly username/password token ko support karta hai.

```text
UsernamePasswordAuthenticationToken
             ↓
DaoAuthenticationProvider.supports()
             ↓
          true
             ↓
authenticate()
```

Then provider user details/password verify karta hai.

---

# 17. Relationship with SecurityContext ⭐⭐⭐⭐⭐

Successful authentication ke baad:

```text
UsernamePasswordAuthenticationToken
              ↓
AuthenticationManager
              ↓
Authenticated Authentication
              ↓
SecurityContext
              ↓
SecurityContextHolder
```

Important distinction:

> Token itself `SecurityContext` nahi hai. Token `Authentication` represent karta hai; SecurityContext us Authentication ko current security state ke part ke roop mein hold karta hai.

---

# 18. `UsernamePasswordAuthenticationFilter` Connection ⭐⭐⭐⭐⭐

Traditional form-login flow mein `UsernamePasswordAuthenticationFilter` username/password request se authentication token create karke authentication manager ko pass kar sakta hai.

Conceptually:

```text
Login Form
   ↓
UsernamePasswordAuthenticationFilter
   ↓
UsernamePasswordAuthenticationToken
   ↓
AuthenticationManager
   ↓
Provider
```

Custom REST login mein aap manually token create kar sakte ho, but actual architecture application configuration par depend karega.

---

# 19. Form Login vs Custom REST Login

### Form login

```text
Browser
 ↓
Login form
 ↓
Spring Security filter
 ↓
UsernamePasswordAuthenticationToken
 ↓
AuthenticationManager
```

### Custom REST login

```text
JSON request
 ↓
Controller / custom authentication filter
 ↓
UsernamePasswordAuthenticationToken
 ↓
AuthenticationManager
```

Same token abstraction use ho sakti hai, but surrounding flow different ho sakta hai.

---

# 20. Credentials Erasure ⭐⭐⭐⭐⭐

Sensitive credentials ko successful authentication ke baad retain karna unnecessary hai.

Spring Security authentication infrastructure credential erasure support karta hai through `CredentialsContainer`-related mechanisms.

Conceptually:

```text
Raw Password
     ↓
Authentication
     ↓
Provider validates
     ↓
Successful Authentication
     ↓
Credentials erased / not retained
```

### Production rule

> **Never log or unnecessarily store raw passwords.**

---

# 21. Authorities and Authenticated Token

Successful authentication ke baad token/principal ke saath authorities available ho sakti hain.

Example:

```text
Principal = UserDetails(nirbhay)
Authorities =
  ROLE_USER
  ORDER_READ
```

Then:

```java
@PreAuthorize("hasAuthority('ORDER_READ')")
```

Authorization layer current Authentication ki authorities evaluate karti hai.

---

# 22. Common Mistake — Constructor Means Trusted Authentication ⭐⭐⭐⭐⭐

A common misconception:

> “Three-argument constructor use kiya, therefore user authenticated hai, so safe hai.”

Not safe as a general security design.

Authenticated representation trusted authentication process se aani chahiye.

```text
Untrusted input
    ↓
❌ Don't directly mark authenticated

Trusted authentication process
    ↓
✅ Authenticated Authentication
```

---

# 23. Common Mistake — Password is Always Available

Wrong assumption:

```java
authentication.getCredentials()
```

hamesha raw password return karega.

After authentication credentials may be erased or not retained.

Therefore:

```text
credentials availability ≠ guaranteed forever
```

---

# 24. Common Mistake — Token = JWT

`UsernamePasswordAuthenticationToken` aur JWT same cheez nahi hain.

```text
UsernamePasswordAuthenticationToken
 = Spring Security Authentication object

JWT
 = signed token format used for carrying claims
```

JWT authentication ke andar different `Authentication` implementation/token use ho sakta hai.

---

# 25. JWT Architecture Distinction ⭐⭐⭐⭐⭐

Username/password login:

```text
Username + Password
      ↓
UsernamePasswordAuthenticationToken
      ↓
AuthenticationManager
```

Bearer JWT request:

```text
Authorization: Bearer JWT
      ↓
Bearer-token authentication infrastructure
      ↓
JWT validation
      ↓
Authentication
```

Isliye har authenticated request ko `UsernamePasswordAuthenticationToken` assume nahi karna chahiye.

---

# 26. Custom Authentication Example

Suppose application custom login service use karti hai:

```text
Mobile Number + OTP
        ↓
CustomAuthenticationToken
        ↓
AuthenticationManager
        ↓
CustomAuthenticationProvider
```

Yahan `UsernamePasswordAuthenticationToken` appropriate token nahi ho sakta.

### Interview point

> Authentication token should represent the authentication mechanism being used.

---

# 27. Security Architecture Summary ⭐⭐⭐⭐⭐

```text
                 Credentials
                     ↓
        UsernamePasswordAuthenticationToken
                     ↓
              AuthenticationManager
                     ↓
                ProviderManager
                     ↓
          AuthenticationProvider
                     ↓
          Authenticated Authentication
                     ↓
               SecurityContext
                     ↓
                Authorization
```

---

# 28. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is UsernamePasswordAuthenticationToken?

> It is an Authentication implementation commonly used to represent username/password authentication requests and authenticated username/password principals.

### Q2. Is it an interface?

> No. It is a concrete class implementing the Authentication contract through Spring Security's authentication token hierarchy.

### Q3. Why is it used?

> It packages username/password authentication information into the standard Authentication abstraction expected by AuthenticationManager.

### Q4. What are its important constructors/factory methods?

> An unauthenticated username/password representation and an authenticated representation with principal, credentials and authorities are supported; modern code can use explicit `unauthenticated(...)` and `authenticated(...)` factory methods.

### Q5. What is `supports()` related to this token?

> AuthenticationProvider uses `supports()` to indicate whether it can handle `UsernamePasswordAuthenticationToken`.

### Q6. What happens after `authenticate()` succeeds?

> An authenticated Authentication is returned, and the surrounding security infrastructure can associate it with the SecurityContext.

### Q7. Is the password always available from `getCredentials()`?

> No. Credentials may be erased after successful authentication or may not be retained.

### Q8. Is this token the same as JWT?

> No. It is a Spring Security Authentication object; JWT is a token format. JWT bearer authentication can use a different Authentication representation.

### Q9. Should we manually create an authenticated token from user input?

> No. Authenticated state should only be established by trusted authentication logic.

### Q10. Which provider commonly supports it for username/password?

> `DaoAuthenticationProvider` commonly handles username/password authentication involving `UsernamePasswordAuthenticationToken`.

---

# 29. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“UsernamePasswordAuthenticationToken is a Spring Security Authentication implementation commonly used for username/password authentication. Before authentication, it represents the username and password as an unauthenticated authentication request. This token is passed to AuthenticationManager, which delegates through ProviderManager to a suitable AuthenticationProvider, commonly DaoAuthenticationProvider. The provider loads user details and verifies the password, then returns an authenticated Authentication. The security infrastructure can associate that Authentication with the SecurityContext. The token also exposes principal, credentials and authorities, but raw credentials should never be logged and may be erased after successful authentication. It is important not to confuse this class with JWT: JWT is a token format, while UsernamePasswordAuthenticationToken is a Spring Security Authentication object.”**

---

# 30. 30-Second Hinglish Answer

> **“UsernamePasswordAuthenticationToken ek Spring Security Authentication implementation hai jo username-password authentication request ko represent karta hai. Usually username aur password se unauthenticated token banta hai, jo AuthenticationManager ko diya jata hai. Manager ProviderManager ke through suitable AuthenticationProvider ko call karta hai. Successful validation ke baad authenticated Authentication milti hai aur security infrastructure usko SecurityContext mein associate karta hai. Raw password ko log ya unnecessarily store nahi karna chahiye.”**

---

# 31. Whiteboard Memory ⭐⭐⭐⭐⭐

```text
Username + Password
        ↓
UsernamePasswordAuthenticationToken
        ↓
AuthenticationManager
        ↓
ProviderManager
        ↓
AuthenticationProvider
        ↓
Authenticated Authentication
        ↓
SecurityContext
        ↓
Authorization
```

### Final memory line

> **Token carries the authentication request → Manager delegates → Provider validates → Authentication becomes available to the SecurityContext.**

---

## Navigation

[← S4.1.10 — AuthenticationProvider](../10-AuthenticationProvider/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.12 — UserDetails and UserDetailsService**
