# S4.1.13 — PasswordEncoder

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. Big Picture ⭐⭐⭐⭐⭐

`PasswordEncoder` Spring Security ka core abstraction hai jo passwords ko **securely encode** karne aur authentication ke time raw password ko stored encoded password ke against verify karne ke liye use hota hai.

```text
User enters raw password
        ↓
PasswordEncoder
        ↓
Encoded password
        ↓
Database

Login:
raw password + stored encoded password
        ↓
PasswordEncoder.matches()
        ↓
true / false
```

### Golden rule

> **Password ko decrypt nahi karna hai. Password hashing/encoding ke through store karna hai aur verification ke liye `matches()` use karna hai.**

---

# 2. What is PasswordEncoder?

`PasswordEncoder` ek Spring Security interface hai.

Conceptually:

```java
public interface PasswordEncoder {
    String encode(CharSequence rawPassword);

    boolean matches(
        CharSequence rawPassword,
        String encodedPassword
    );

    default boolean upgradeEncoding(String encodedPassword) {
        return false;
    }
}
```

Important methods:

| Method | Purpose |
|---|---|
| `encode()` | Raw password ko encoded representation mein convert karta hai |
| `matches()` | Raw password ko stored encoded password ke against verify karta hai |
| `upgradeEncoding()` | Existing encoded password ko stronger encoding mein migrate karna hai ya nahi batata hai |

---

# 3. Encoding vs Encryption ⭐⭐⭐⭐⭐

Interview mein ye distinction bahut important hai.

### Encryption

```text
Plaintext
   ↓
Encryption + Key
   ↓
Ciphertext
   ↓
Decryption + Key
   ↓
Plaintext
```

Encryption reversible hoti hai.

### Password hashing

```text
Password
   ↓
Password Hash Function
   ↓
Encoded/hashed representation
```

Password verification mein original password recover nahi kiya jata.

Instead:

```text
raw password
      ↓
PasswordEncoder.matches()
      ↓
stored encoded password
```

### Interview line

> **Passwords should be stored using a one-way password hashing algorithm, not reversible encryption.**

---

# 4. Why PasswordEncoder? ⭐⭐⭐⭐⭐

Suppose user enters:

```text
MyPassword123
```

Database mein ye store karna dangerous hai:

```text
password = MyPassword123
```

Correct approach:

```text
MyPassword123
      ↓
PasswordEncoder
      ↓
encoded representation
      ↓
Database
```

Agar database leak ho jaye, raw passwords directly expose nahi hone chahiye.

---

# 5. `encode()` Method

Example:

```java
String encoded = passwordEncoder.encode("MyPassword123");
```

Result ko exact fixed string assume nahi karna chahiye.

```text
raw password
     ↓
encode()
     ↓
encoded password
```

### Important

Password encoders generally use salt/randomization, so repeated encoding of the same password may produce different encoded values.

---

# 6. `matches()` Method ⭐⭐⭐⭐⭐

Login ke time:

```java
boolean valid = passwordEncoder.matches(
    rawPassword,
    storedEncodedPassword
);
```

Example:

```java
String stored = passwordEncoder.encode("secret");

boolean result = passwordEncoder.matches(
    "secret",
    stored
);

System.out.println(result); // true
```

Wrong password:

```java
passwordEncoder.matches("wrong", stored);
```

Result:

```text
false
```

---

# 7. Why NOT `encode().equals()`? ⭐⭐⭐⭐⭐

Wrong pattern:

```java
passwordEncoder.encode(rawPassword)
        .equals(storedPassword);
```

Why?

Because secure password encoders may use a unique salt/randomization for each encoding.

So:

```text
encode("secret") → encoded-A
encode("secret") → encoded-B
```

Both can represent the same password while being different strings.

Correct:

```java
passwordEncoder.matches(
    "secret",
    storedPassword
);
```

### Interview answer

> **Never compare a newly encoded password with the stored string using `equals()`. Use `matches()` because the stored representation contains the information required for verification.**

---

# 8. PasswordEncoder and Salt ⭐⭐⭐⭐⭐

Modern password hashing algorithms commonly use salts.

Conceptually:

```text
Password
   +
Random Salt
   ↓
Password Hash
```

Stored representation can contain algorithm-specific parameters and salt information.

This helps protect against:

- Rainbow-table attacks
- Precomputed hash attacks
- Identical passwords producing identical stored hashes in systems using per-password random salts

---

# 9. BCryptPasswordEncoder ⭐⭐⭐⭐⭐

A commonly used Spring Security password encoder is:

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

Usage:

```java
String encoded = passwordEncoder.encode("secret");
```

BCrypt is designed for password hashing and includes a work factor/cost parameter.

---

# 10. BCrypt Cost Factor

Example:

```java
new BCryptPasswordEncoder(12);
```

Higher work factor generally means more computation per password operation.

Conceptually:

```text
Cost ↑
  ↓
Computation ↑
  ↓
Attacker's brute-force cost ↑
```

But:

> **Cost should be chosen based on measured application/server performance and current security requirements, not blindly set to the largest value.**

---

# 11. DelegatingPasswordEncoder ⭐⭐⭐⭐⭐

Modern Spring Security applications commonly use `DelegatingPasswordEncoder`.

Example:

```java
PasswordEncoder encoder =
    PasswordEncoderFactories.createDelegatingPasswordEncoder();
```

Stored password representation commonly contains an algorithm identifier/prefix, for example:

```text
{bcrypt}encoded-value
```

Conceptually:

```text
{algorithm-id}encoded-password
       ↓
DelegatingPasswordEncoder
       ↓
Select matching encoder
```

This allows an application to support multiple encoding schemes while having a preferred/default encoder for new passwords.

---

# 12. Why DelegatingPasswordEncoder? ⭐⭐⭐⭐⭐

Real production systems may need password-hash migration.

Example:

```text
Old users → old password encoding
New users → stronger encoding
```

Without a migration strategy, changing algorithms can force password resets.

Delegating approach allows algorithm identification and gradual migration strategies.

---

# 13. Password Encoding in Spring Boot

Typical configuration:

```java
@Configuration
public class SecurityConfig {

    @Bean
    PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories
                .createDelegatingPasswordEncoder();
    }
}
```

Then inject it:

```java
@Service
public class UserService {

    private final PasswordEncoder passwordEncoder;

    public UserService(PasswordEncoder passwordEncoder) {
        this.passwordEncoder = passwordEncoder;
    }
}
```

---

# 14. Registration Flow ⭐⭐⭐⭐⭐

When creating a user:

```java
String encodedPassword =
    passwordEncoder.encode(request.password());

user.setPassword(encodedPassword);
userRepository.save(user);
```

Flow:

```text
Registration Request
       ↓
Raw Password
       ↓
PasswordEncoder.encode()
       ↓
Encoded Password
       ↓
Database
```

### Critical rule

> **Never save the raw password.**

---

# 15. Login Flow ⭐⭐⭐⭐⭐

Typical username/password authentication:

```text
Login Request
     ↓
UsernamePasswordAuthenticationToken
     ↓
AuthenticationManager
     ↓
DaoAuthenticationProvider
     ↓
UserDetailsService
     ↓
UserDetails
     ↓
PasswordEncoder.matches()
     ↓
Authenticated Authentication
```

Usually application code should let the configured authentication provider perform this verification rather than manually duplicating password checks in controllers.

---

# 16. DaoAuthenticationProvider + PasswordEncoder ⭐⭐⭐⭐⭐

Relationship:

```text
DaoAuthenticationProvider
        ↓
UserDetailsService
        ↓
UserDetails
        ↓
PasswordEncoder
        ↓
Password verification
```

The provider obtains the stored password from `UserDetails` and uses the configured `PasswordEncoder` to verify the presented password.

---

# 17. PasswordEncoder vs UserDetailsService

| Component | Responsibility |
|---|---|
| `UserDetailsService` | User details load karta hai |
| `UserDetails` | Security user data represent karta hai |
| `PasswordEncoder` | Password encode/verify karta hai |
| `AuthenticationProvider` | Authentication perform karta hai |
| `AuthenticationManager` | Authentication providers ko coordinate karta hai |

Memory:

```text
UserDetailsService → User ka data lao
PasswordEncoder    → Password verify karo
AuthenticationProvider → Authentication complete karo
```

---

# 18. NoOpPasswordEncoder ⭐⭐⭐⭐⭐

Spring Security historically provides `NoOpPasswordEncoder`, but plaintext password storage is not appropriate for production.

Conceptually:

```text
raw password
    ↓
NoOp
    ↓
same raw password
```

### Use

Only controlled demos/testing scenarios where the security implications are fully understood.

### Production

❌ Avoid.

---

# 19. `PasswordEncoderFactories.createDelegatingPasswordEncoder()`

Example:

```java
@Bean
PasswordEncoder passwordEncoder() {
    return PasswordEncoderFactories
        .createDelegatingPasswordEncoder();
}
```

Advantages:

- Delegation based on encoded password id
- Supports multiple encoders
- Provides a modern migration-friendly architecture
- New passwords use the configured default encoder

Exact defaults can change across Spring Security versions, so don't memorize a specific default algorithm solely from older tutorials.

---

# 20. Password Hash Migration ⭐⭐⭐⭐⭐

Suppose old hashes use an older algorithm:

```text
Old User
   ↓
Old Hash
```

Application can gradually migrate:

```text
Login
 ↓
Verify old hash
 ↓
Authentication successful
 ↓
Encode password using stronger/current encoder
 ↓
Update DB
```

`upgradeEncoding()` can help an application determine whether an encoded password should be upgraded.

---

# 21. `upgradeEncoding()` ⭐⭐⭐⭐

Conceptually:

```java
if (passwordEncoder.upgradeEncoding(storedPassword)) {
    // re-encode after successful authentication
}
```

Important:

> `upgradeEncoding()` itself does not authenticate the user. It indicates whether an existing encoded representation is considered outdated according to the encoder's policy.

A common migration pattern is:

```text
Authenticate successfully
        ↓
Check upgradeEncoding()
        ↓
If required
        ↓
Re-encode raw password
        ↓
Update stored password
```

---

# 22. Password Change Flow ⭐⭐⭐⭐⭐

When user changes password:

```text
Current password verification
          ↓
New password
          ↓
PasswordEncoder.encode()
          ↓
Store encoded new password
```

Do not store:

```text
newPassword = raw password
```

---

# 23. Forgot Password Flow ⭐⭐⭐⭐⭐

Typical secure flow:

```text
Forgot password request
       ↓
Short-lived reset token
       ↓
User verifies reset token
       ↓
New password
       ↓
PasswordEncoder.encode()
       ↓
Store new encoded password
```

The reset token itself is a separate credential mechanism; don't confuse it with the password hash.

---

# 24. PasswordEncoder and JWT ⭐⭐⭐⭐⭐

PasswordEncoder and JWT solve different problems.

```text
PasswordEncoder
    ↓
Protect stored password

JWT
    ↓
Carry authentication/authorization claims after authentication
```

Typical login:

```text
Username + Password
       ↓
PasswordEncoder verification
       ↓
Authentication success
       ↓
JWT issued
```

Subsequent request:

```text
JWT
 ↓
JWT validation
 ↓
Authentication
```

PasswordEncoder generally isn't needed to verify the JWT itself.

---

# 25. PasswordEncoder and Database ⭐⭐⭐⭐⭐

Database example:

```text
users
---------------------------------
id | username | password
---------------------------------
1  | nirbhay  | {bcrypt}...
```

Never:

```text
password = MyPassword123
```

Use an appropriate column size/type for the selected encoder and leave room for future algorithm/hash migration.

---

# 26. Password Hashing vs General Hashing ⭐⭐⭐⭐

Don't use a fast general-purpose hash like plain SHA-256 alone for password storage.

Why?

```text
Fast hash
   ↓
Very high guesses/second
   ↓
Brute-force becomes easier
```

Password hashing algorithms are intentionally computationally expensive and designed for password storage.

Examples in Spring Security ecosystem include:

- BCrypt
- PBKDF2
- SCrypt
- Argon2

Exact algorithm choice should follow current security requirements and organizational standards.

---

# 27. Argon2 / PBKDF2 / SCrypt / BCrypt ⭐⭐⭐⭐

| Encoder | Main idea |
|---|---|
| BCrypt | Adaptive password hashing with configurable cost |
| PBKDF2 | Iterated key-derivation/password hashing |
| SCrypt | Memory-hard password hashing |
| Argon2 | Modern memory-hard password hashing family |

### Interview point

Don't answer:

> “BCrypt is always the only correct encoder.”

Better:

> **Use a modern adaptive password hashing algorithm appropriate for the application's security and operational requirements; Spring Security supports multiple options.**

---

# 28. Why Salt Matters ⭐⭐⭐⭐⭐

Without unique salts:

```text
password A → same hash
password A → same hash
```

An attacker can identify users with the same password more easily and benefit from precomputed attacks.

With unique salts:

```text
password A + salt1 → hash1
password A + salt2 → hash2
```

Same password can have different stored representations.

---

# 29. Pepper — Advanced Interview Topic ⭐⭐⭐

A pepper is an additional secret value kept outside the database, usually managed separately from stored password hashes.

Conceptually:

```text
Password + Salt + Pepper
       ↓
Password Hash
```

Pepper is an advanced defense and adds operational complexity. If used, secret-management and rotation/recovery strategy must be designed carefully.

---

# 30. Timing and Error Handling ⭐⭐⭐⭐

Authentication systems should avoid creating obvious user-enumeration differences.

For example, externally exposing:

```text
User doesn't exist
```

versus:

```text
Password is wrong
```

can reveal whether an account exists.

Use framework-supported authentication failure handling and return appropriate generic external messages where required.

---

# 31. PasswordEncoder Security Rules ⭐⭐⭐⭐⭐

### Never do

```java
user.setPassword(rawPassword);
```

### Do

```java
user.setPassword(
    passwordEncoder.encode(rawPassword)
);
```

### Never do

```java
passwordEncoder.encode(raw).equals(stored);
```

### Do

```java
passwordEncoder.matches(raw, stored);
```

### Never do

```java
log.info("password={}", rawPassword);
```

### Do

Keep passwords out of logs, traces, events and error responses.

---

# 32. Common Interview Traps ⭐⭐⭐⭐⭐

### Trap 1 — PasswordEncoder encrypts password

❌ Not the right mental model.

> It provides password encoding/hashing and matching; passwords are not expected to be reversibly decrypted.

### Trap 2 — `encode()` twice and compare

❌ Wrong.

> Use `matches()`.

### Trap 3 — Same password always creates same encoded string

❌ Don't assume that.

> Salting/randomization can make encoded outputs different.

### Trap 4 — SHA-256 alone is enough

❌ Not recommended for password storage.

> Use a password-specific adaptive hashing algorithm.

### Trap 5 — PasswordEncoder validates JWT

❌ No.

> JWT validation is a separate authentication mechanism.

### Trap 6 — NoOpPasswordEncoder is production safe

❌ No.

> Plaintext-like storage is inappropriate for production.

### Trap 7 — Higher cost is always better

❌ Not blindly.

> Benchmark and choose an appropriate cost for your infrastructure and security requirements.

---

# 33. Production Architecture ⭐⭐⭐⭐⭐

```text
                    Registration
                         ↓
                    Raw Password
                         ↓
                 PasswordEncoder
                         ↓
                  Encoded Password
                         ↓
                      Database

Login:

Raw Password ───────────────┐
                            ↓
                     AuthenticationProvider
                            ↓
                      UserDetailsService
                            ↓
                        UserDetails
                            ↓
                     PasswordEncoder
                            ↓
                         matches()
                            ↓
                  Authenticated Authentication
                            ↓
                       SecurityContext
```

---

# 34. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is PasswordEncoder?

> It is Spring Security's abstraction for encoding passwords and verifying raw passwords against stored encoded representations.

### Q2. Difference between `encode()` and `matches()`?

> `encode()` creates a stored representation from a raw password. `matches()` verifies a raw password against an existing encoded representation.

### Q3. Why not use `encode(raw).equals(stored)`?

> Because password encoders can use salts/randomization, so the same raw password may produce different encoded strings. `matches()` performs the correct verification.

### Q4. Is PasswordEncoder encryption?

> No. Password storage should use one-way password hashing/encoding rather than reversible encryption.

### Q5. What is BCryptPasswordEncoder?

> It is a Spring Security PasswordEncoder implementation based on BCrypt, an adaptive password hashing algorithm with a configurable work factor.

### Q6. What is DelegatingPasswordEncoder?

> It delegates password matching to an encoder selected using an algorithm identifier in the stored encoded value and supports migration between encoding schemes.

### Q7. Why is salt important?

> A unique salt makes identical passwords produce different stored representations and makes precomputed attacks harder.

### Q8. What is `upgradeEncoding()`?

> It indicates whether an existing encoded password should be upgraded according to the encoder's current policy.

### Q9. Should raw passwords be stored in DB?

> Never. Store only appropriate password hashes/encoded representations.

### Q10. How does PasswordEncoder fit with DaoAuthenticationProvider?

> DaoAuthenticationProvider loads UserDetails through UserDetailsService and uses PasswordEncoder to verify the presented password against the stored password.

### Q11. Does PasswordEncoder verify JWT?

> No. PasswordEncoder is for password storage/verification; JWT validation is handled by the JWT authentication infrastructure.

### Q12. Which password encoder should we use?

> Choose a current adaptive password hashing algorithm supported by the application's security requirements and operational constraints; BCrypt, PBKDF2, SCrypt and Argon2 are examples available in the Spring Security ecosystem.

---

# 35. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“PasswordEncoder is Spring Security's abstraction for securely encoding passwords and verifying them during authentication. We use `encode()` when storing or changing a password, and `matches()` when validating a login password against the stored encoded value. We should never compare a newly encoded password using `equals()` because password encoders use salting and the same password can produce different encoded representations. PasswordEncoder is not reversible encryption; password storage should use an appropriate adaptive password hashing algorithm. BCrypt is a common option, while Spring Security also supports other algorithms such as PBKDF2, SCrypt and Argon2. In a typical username/password flow, DaoAuthenticationProvider loads UserDetails through UserDetailsService and uses PasswordEncoder to verify the supplied password. DelegatingPasswordEncoder can support algorithm identifiers and gradual password-hash migration. PasswordEncoder is separate from JWT validation.”**

---

# 36. 30-Second Hinglish Answer

> **“PasswordEncoder Spring Security ka abstraction hai jo password ko securely encode aur verify karta hai. Registration ya password change ke time `encode()` use hota hai, aur login ke time `matches(rawPassword, storedPassword)` use hota hai. `encode().equals()` nahi karna chahiye because salt ki wajah se same password ke encoded values different ho sakte hain. PasswordEncoder encryption nahi hai; password ko one-way adaptive hashing ke through store karte hain. BCrypt common option hai, aur Spring Security multiple algorithms support karta hai.”**

---

# 37. Whiteboard Memory ⭐⭐⭐⭐⭐

```text
REGISTER
Raw Password
     ↓
PasswordEncoder.encode()
     ↓
Encoded Password
     ↓
DB

LOGIN
Raw Password
     ↓
AuthenticationProvider
     ↓
UserDetailsService
     ↓
Stored Encoded Password
     ↓
PasswordEncoder.matches()
     ↓
Authentication Success / Failure
```

### Final memory line

> **`encode()` → store password safely | `matches()` → verify password | Never decrypt / never compare `encode()` output with `equals()`.**

---

## Navigation

[← S4.1.12 — UserDetails and UserDetailsService](../12-UserDetails-and-UserDetailsService/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.14 — GrantedAuthority and Roles**
