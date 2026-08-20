# S4.1.12 — UserDetails and UserDetailsService

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. Big Picture ⭐⭐⭐⭐⭐

`UserDetails` aur `UserDetailsService` username/password authentication flow ke core abstractions hain.

```text
Authentication Request
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
```

### Memory trick

```text
UserDetails        = User ka security data
UserDetailsService = UserDetails load karne wali service
```

---

# 2. What is UserDetails?

`UserDetails` Spring Security ka interface hai jo authenticated user's information ko represent karta hai.

Typical contract:

```java
public interface UserDetails extends Serializable {

    Collection<? extends GrantedAuthority> getAuthorities();

    String getPassword();

    String getUsername();

    boolean isAccountNonExpired();

    boolean isAccountNonLocked();

    boolean isCredentialsNonExpired();

    boolean isEnabled();
}
```

### Important

`UserDetails` ka purpose application ke complete business user object ko represent karna nahi hai. Ye **security-related user information** provide karta hai.

---

# 3. UserDetails ke Main Methods ⭐⭐⭐⭐⭐

| Method | Meaning |
|---|---|
| `getUsername()` | User identity/login identifier |
| `getPassword()` | Stored password representation, normally encoded |
| `getAuthorities()` | Roles/permissions |
| `isAccountNonExpired()` | Account expired hai ya nahi |
| `isAccountNonLocked()` | Account locked hai ya nahi |
| `isCredentialsNonExpired()` | Credentials expired hain ya nahi |
| `isEnabled()` | Account enabled hai ya nahi |

Memory:

```text
UserDetails
 ├── Identity
 ├── Password
 ├── Authorities
 └── Account Status
```

---

# 4. What is UserDetailsService?

`UserDetailsService` ek interface hai jo username ke basis par `UserDetails` load karta hai.

```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username)
            throws UsernameNotFoundException;
}
```

### Simple meaning

> **UserDetailsService = security user information ka loader.**

---

# 5. UserDetails vs UserDetailsService ⭐⭐⭐⭐⭐

| | UserDetails | UserDetailsService |
|---|---|---|
| Type | Interface/model abstraction | Service interface |
| Represents | User's security information | UserDetails load karne ka mechanism |
| Main method | `getUsername()`, `getAuthorities()`, etc. | `loadUserByUsername()` |
| Responsibility | User data expose karna | User data retrieve karna |
| Database call | Not necessarily | Usually implementation mein hoti hai |

### Interview line

> **UserDetails represents the user's security information, while UserDetailsService loads that information by username.**

---

# 6. Complete DAO Authentication Flow ⭐⭐⭐⭐⭐

```text
Username + Password
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
loadUserByUsername(username)
        ↓
UserDetails
        ↓
PasswordEncoder
        ↓
Authenticated Authentication
        ↓
SecurityContext
```

### Most important relationship

```text
DaoAuthenticationProvider
          ↓
   UserDetailsService
          ↓
      UserDetails
```

---

# 7. Why UserDetailsService? ⭐⭐⭐⭐⭐

Spring Security ko application ke database schema ka knowledge nahi hona chahiye.

For example DB table:

```text
users
--------------------------------
id | login | pwd_hash | status
```

Spring Security ko directly ye assume nahi karna chahiye ki columns `username`, `password`, `enabled` hi honge.

Instead:

```text
Database
   ↓
Application Repository / DAO
   ↓
UserDetailsService
   ↓
UserDetails
   ↓
Spring Security
```

This creates separation between **application data model** and **security abstraction**.

---

# 8. Default UserDetails Implementation — User

Spring Security `User` class provide karta hai as a convenient `UserDetails` implementation.

Conceptual example:

```java
UserDetails user = User.withUsername("nirbhay")
        .password(passwordEncoder.encode("secret"))
        .roles("USER")
        .build();
```

Important:

```text
roles("USER")
      ↓
ROLE_USER
```

`roles(...)` internally role prefix convention ke according authorities create karta hai.

If you want an exact authority:

```java
.authorities("ORDER_READ")
```

---

# 9. In-Memory UserDetailsService ⭐⭐⭐⭐⭐

Learning/testing ke liye:

```java
@Bean
UserDetailsService users(PasswordEncoder passwordEncoder) {

    UserDetails user = User.withUsername("nirbhay")
            .password(passwordEncoder.encode("secret"))
            .roles("USER")
            .build();

    return new InMemoryUserDetailsManager(user);
}
```

Flow:

```text
InMemoryUserDetailsManager
          ↓
UserDetails
```

Production applications mein user data generally persistent store se aata hai.

---

# 10. Database-backed UserDetailsService ⭐⭐⭐⭐⭐

Production example:

```java
@Service
public class CustomUserDetailsService
        implements UserDetailsService {

    private final UserRepository userRepository;

    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username)
            throws UsernameNotFoundException {

        AppUser user = userRepository.findByUsername(username)
                .orElseThrow(() ->
                    new UsernameNotFoundException(
                        "User not found"));

        return User.withUsername(user.getUsername())
                .password(user.getPasswordHash())
                .authorities(user.getAuthorities())
                .accountExpired(!user.isAccountActive())
                .build();
    }
}
```

Actual account-state mapping application requirements ke according design karna chahiye.

---

# 11. Repository + UserDetailsService Architecture ⭐⭐⭐⭐⭐

```text
Controller / Security Filter
          ↓
AuthenticationManager
          ↓
DaoAuthenticationProvider
          ↓
UserDetailsService
          ↓
UserRepository
          ↓
Database
```

Return path:

```text
Database
   ↓
AppUser
   ↓
UserDetailsService
   ↓
UserDetails
   ↓
DaoAuthenticationProvider
```

---

# 12. UserDetailsService Does NOT Authenticate Password ⭐⭐⭐⭐⭐

Very important interview point.

`UserDetailsService`:

```text
loads user details
```

It does not generally:

```text
compare raw password manually
```

Username/password authentication mein `DaoAuthenticationProvider` loaded `UserDetails` aur configured `PasswordEncoder` ko use karke password validation karta hai.

### Correct architecture

```text
UserDetailsService
      ↓
Load UserDetails
      ↓
DaoAuthenticationProvider
      ↓
PasswordEncoder
      ↓
Password validation
```

---

# 13. Password Handling ⭐⭐⭐⭐⭐

Database mein raw password store nahi karna chahiye.

```text
User enters:
secret

Database stores:
encoded password hash
```

Provider conceptually:

```java
passwordEncoder.matches(
    rawPassword,
    userDetails.getPassword()
);
```

### Rule

> **Encode when storing/updating the password; use `matches()` during authentication.**

Don't do:

```java
passwordEncoder.encode(rawPassword)
    .equals(storedPassword)
```

as a general password verification pattern, because adaptive password encoders may use salts and should be verified with `matches()`.

---

# 14. UserDetails Authorities ⭐⭐⭐⭐⭐

`UserDetails.getAuthorities()` authorization ke liye authorities provide karta hai.

Example:

```text
ROLE_USER
ORDER_READ
ORDER_WRITE
```

Then:

```java
@PreAuthorize("hasAuthority('ORDER_READ')")
```

or role-based:

```java
@PreAuthorize("hasRole('USER')")
```

### Role vs Authority

```text
roles("USER")
      ↓
ROLE_USER
```

`hasRole("USER")` commonly `ROLE_` prefix convention ke saath evaluate hota hai, while `hasAuthority("ROLE_USER")` exact authority value check kar sakta hai.

---

# 15. Account Status Flags ⭐⭐⭐⭐⭐

`UserDetails` ke status methods authentication ko additional account rules check karne dete hain.

```java
isAccountNonExpired()
isAccountNonLocked()
isCredentialsNonExpired()
isEnabled()
```

Conceptual flow:

```text
UserDetails
   ↓
Account status checks
   ├── expired?
   ├── locked?
   ├── credentials expired?
   └── enabled?
   ↓
Authentication allowed / rejected
```

---

# 16. UsernameNotFoundException ⭐⭐⭐⭐⭐

Agar user nahi milta:

```java
throw new UsernameNotFoundException(
    "User not found"
);
```

Ye `AuthenticationException` hierarchy ka part hai.

Security-sensitive applications mein user-enumeration risk ko consider karna chahiye; externally overly specific login errors expose karna avoid kiya ja sakta hai.

---

# 17. Why Return UserDetails Instead of Entity?

Suppose application entity:

```java
@Entity
class AppUser {
    Long id;
    String email;
    String passwordHash;
    String department;
    String phone;
}
```

Security ko sab fields ki need nahi.

```text
AppUser
   ↓
UserDetails
   ├── username
   ├── password
   ├── authorities
   └── account status
```

Benefits:

- Security abstraction separate
- Sensitive/business fields expose nahi hote
- Authentication code cleaner
- Application model aur security model decoupled

---

# 18. Custom UserDetails ⭐⭐⭐⭐

Complex application mein custom `UserDetails` implement kar sakte ho.

```java
public class CustomUserDetails implements UserDetails {

    private final AppUser user;

    public CustomUserDetails(AppUser user) {
        this.user = user;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return user.getAuthorities();
    }

    @Override
    public String getPassword() {
        return user.getPasswordHash();
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return !user.isLocked();
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return user.isEnabled();
    }
}
```

---

# 19. When Custom UserDetails is Useful?

Useful when authenticated principal ko extra application-specific information expose karni ho.

Example:

```text
CustomUserDetails
 ├── username
 ├── password
 ├── authorities
 ├── userId
 └── tenantId
```

Then application code:

```java
CustomUserDetails user =
    (CustomUserDetails) authentication.getPrincipal();
```

Type cast tabhi karo jab actual principal type known/configured ho.

---

# 20. `UserDetails` is not Your Complete User Entity ⭐⭐⭐⭐⭐

Interview mein bolna:

> **UserDetails is a security representation of a user, not necessarily the application's complete domain entity.**

Example:

```text
Domain User
 ├── id
 ├── name
 ├── email
 ├── address
 ├── paymentProfile
 └── preferences

Security UserDetails
 ├── username
 ├── password
 ├── authorities
 └── account status
```

---

# 21. `UserDetailsService` and Caching ⭐⭐⭐⭐

Database-backed applications mein repeated authentication lookups expensive ho sakte hain.

Possible architecture:

```text
Authentication
     ↓
UserDetailsService
     ↓
Cache
  ↙     ↘
Hit     Miss
 ↓       ↓
UserDetails Repository/DB
```

Spring Security ecosystem caching support provide karta hai, but caching strategy application requirements ke according choose karni chahiye.

Account lock/disable changes ke baad stale cache ko handle karna important hai.

---

# 22. Stateless JWT and UserDetails ⭐⭐⭐⭐⭐

JWT resource-server request mein username/password login ke jaise `UserDetailsService` necessarily involved ho, ye assume nahi karna chahiye.

Conceptually:

```text
Username/Password login
        ↓
UserDetailsService
        ↓
UserDetails
```

While JWT resource-server:

```text
Bearer JWT
    ↓
JWT validation
    ↓
JWT claims → Authentication/principal
    ↓
SecurityContext
```

Agar application ko token claims se beyond database user information chahiye, custom authority/principal mapping ya additional application lookup ho sakta hai.

---

# 23. UserDetailsService vs JWT ⭐⭐⭐⭐⭐

Interview question:

**“JWT use kar rahe hain, kya UserDetailsService har request par database hit karega?”**

Correct answer:

> **Not necessarily.** In a JWT resource-server setup, the token can be validated and converted into an Authentication without using UserDetailsService for every request. Whether a database lookup occurs depends on the application's design and custom requirements.

---

# 24. `UserDetailsService` Bean Discovery

Spring Security application configured `UserDetailsService` bean ko authentication infrastructure mein use kar sakta hai, especially username/password authentication configuration ke context mein.

Example:

```java
@Bean
UserDetailsService userDetailsService() {
    return username -> repository.findByUsername(username)
        .orElseThrow(() ->
            new UsernameNotFoundException("User not found"));
}
```

Exact auto-configuration behavior Spring Boot version aur security configuration par depend kar sakta hai.

---

# 25. Multiple UserDetailsService Implementations ⭐⭐⭐⭐

Multiple services ho sakti hain, but authentication architecture ko clearly define karna important hai.

Example:

```text
Provider A → UserDetailsService A
Provider B → UserDetailsService B
```

Don't rely on accidental bean selection when multiple authentication sources exist; configure the providers explicitly where required.

---

# 26. Security Architecture ⭐⭐⭐⭐⭐

```text
                    Login Request
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
                  ↙          ↘
          PasswordEncoder   Authorities
                  ↓              ↓
             Authentication → Authorization
                         ↓
                   SecurityContext
```

---

# 27. Common Interview Traps ⭐⭐⭐⭐⭐

### Trap 1 — UserDetailsService authenticates user

❌ Not precise.

> It loads UserDetails. Provider performs authentication.

### Trap 2 — UserDetails is database entity

❌ Not necessarily.

> It is a security representation/abstraction.

### Trap 3 — UserDetailsService always queries DB

❌ Not necessarily.

> It can load from DB, LDAP, external service, cache, etc.

### Trap 4 — UserDetailsService validates password

❌ Not its primary responsibility.

> DaoAuthenticationProvider + PasswordEncoder handle username/password verification.

### Trap 5 — JWT always needs UserDetailsService

❌ Wrong.

> JWT resource-server authentication can derive Authentication from a validated token.

### Trap 6 — Password stored as plaintext

❌ Never.

> Store encoded password hashes and verify with `PasswordEncoder.matches()`.

---

# 28. Production Best Practices ⭐⭐⭐⭐⭐

- Raw passwords kabhi store/log mat karo.
- Password verification ke liye `PasswordEncoder.matches()` use karo.
- UserDetails ko minimum required security information tak rakho.
- Authorities ko trusted source se derive karo.
- Account status flags correctly map karo.
- Sensitive fields ko authentication principal mein unnecessarily expose mat karo.
- User enumeration risk consider karo.
- Cache use karte waqt account-status invalidation consider karo.
- JWT architecture mein unnecessary DB lookup avoid karo.
- Multiple authentication providers/sources ko explicit architecture ke saath configure karo.

---

# 29. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is UserDetails?

> UserDetails is Spring Security's abstraction representing security-related information about an authenticated user.

### Q2. What is UserDetailsService?

> It is a service interface used to load UserDetails by username.

### Q3. What is the main method of UserDetailsService?

```java
loadUserByUsername(String username)
```

### Q4. Does UserDetailsService authenticate the password?

> No. In the common DAO username/password flow, it loads UserDetails; DaoAuthenticationProvider and PasswordEncoder perform the authentication checks.

### Q5. What does UserDetails contain?

> Username, password representation, authorities and account-status information.

### Q6. Can UserDetails be a custom class?

> Yes, a custom class can implement UserDetails when application-specific principal information is needed.

### Q7. Can UserDetailsService load from sources other than DB?

> Yes. It can load from any source such as LDAP, external service, cache or another persistence mechanism.

### Q8. What happens if user is not found?

> The implementation commonly throws UsernameNotFoundException.

### Q9. Why use UserDetails instead of returning the entity directly?

> To decouple Spring Security's security model from the application's domain model.

### Q10. Does JWT authentication always call UserDetailsService?

> No. JWT resource-server authentication can establish Authentication from validated token claims without a UserDetailsService lookup on every request.

### Q11. Difference between UserDetails and Authentication?

> UserDetails represents loaded security information about the user, while Authentication represents the current authentication request/result and also carries authorities and authentication state.

### Q12. Difference between UserDetailsService and AuthenticationProvider?

> UserDetailsService loads user information; AuthenticationProvider uses the loaded information and credentials/token to perform authentication.

---

# 30. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“UserDetails is Spring Security's security representation of a user. It provides username, encoded password, authorities and account-status information such as enabled, locked and expired states. UserDetailsService is an interface whose main method is `loadUserByUsername`, and its responsibility is to load UserDetails from a source such as a database, LDAP or external service. In the common username/password flow, DaoAuthenticationProvider calls UserDetailsService, receives UserDetails, and uses PasswordEncoder to verify the supplied password. UserDetailsService itself is not the password authentication mechanism. This abstraction also decouples the application's domain user entity from Spring Security. In JWT resource-server applications, UserDetailsService is not necessarily called for every request because authentication can be established from a validated token.”**

---

# 31. 30-Second Hinglish Answer

> **“UserDetails user ki security information ko represent karta hai—username, encoded password, authorities aur account status. UserDetailsService ka kaam username ke basis par UserDetails load karna hai. Username-password flow mein DaoAuthenticationProvider UserDetailsService se user load karta hai aur PasswordEncoder se password verify karta hai. UserDetailsService khud authentication nahi karta. JWT setup mein har request par UserDetailsService call hona necessary nahi hai.”**

---

# 32. Whiteboard Memory ⭐⭐⭐⭐⭐

```text
Username + Password
        ↓
Authentication Token
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
   ┌────┼───────────┐
   ↓    ↓           ↓
Username Password Authorities
        ↓
   PasswordEncoder
        ↓
Authenticated Authentication
        ↓
SecurityContext
```

### Final memory line

> **UserDetails = security user data → UserDetailsService = loads that data → AuthenticationProvider = uses it to authenticate.**

---

## Navigation

[← S4.1.11 — UsernamePasswordAuthenticationToken](../11-UsernamePasswordAuthenticationToken/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.13 — PasswordEncoder**
