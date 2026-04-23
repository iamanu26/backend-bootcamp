# The Ultimate Guide to Authentication & Authorization for Backend Engineers

A deep-dive technical guide covering identity management, security protocols, and architectural patterns for modern web systems.

---

## 1. Fundamental Definitions

Before diving into implementations, it is critical to distinguish between the two pillars of security:

* **Authentication (AuthN):** Validates **Identity**. Answers: *"Are you who you say you are?"* (e.g., Logging in with a password).
* **Authorization (AuthZ):** Validates **Permissions**. Answers: *"Are you allowed to do this specific action?"* (e.g., Can a user delete a post?).

---

## 2. Authentication Architectures

### A. Session-Based (Stateful) Authentication
This is the traditional method used for monoliths and standard web apps.

* **The Workflow:**
    1.  User sends credentials (Username/Password).
    2.  Server verifies and creates a **Session ID**.
    3.  Server stores this ID in memory (Redis) or a Database.
    4.  Server sends the ID back to the browser via a **Set-Cookie** header.
    5.  Browser automatically includes the cookie in every subsequent request.
* **Pros:** Easy to revoke (just delete the session from the DB), high security.
* **Cons:** Hard to scale (requires a centralized session store), high memory overhead for large user bases.



### B. Token-Based (Stateless) Authentication (JWT)
The modern standard for SPAs, Mobile Apps, and Microservices.

* **The Workflow:**
    1.  User logs in.
    2.  Server signs a **JSON Web Token (JWT)** using a secret key.
    3.  Server sends the JWT to the client.
    4.  Client stores it (LocalStorage or Cookie) and sends it in the `Authorization: Bearer <token>` header.
    5.  Server verifies the signature without checking the database.
* **Structure of a JWT:**
    * **Header:** Metadata like `alg` (algorithm) and `typ` (token type).
    * **Payload:** Contains "Claims" like `sub` (User ID), `role`, and `iat` (Issued At).
    * **Signature:** The hash of the Header + Payload + Secret.
* **Pros:** Highly scalable, works across different domains/services.
* **Cons:** Hard to revoke (valid until expiry), Payload is visible (never store secrets inside).



---

## 3. Delegated Authorization (OAuth 2.0 & OIDC)

### OAuth 2.0: The Authorization Framework
OAuth is not a login protocol; it is a **delegation** protocol. It allows an app (Client) to access data on another platform (Resource Server) on behalf of a user.

* **Key Grant Types:**
    1.  **Authorization Code Flow:** Used for server-side apps (most secure).
    2.  **Client Credentials Flow:** Used for Machine-to-Machine communication (no user involved).
    3.  **Device Code Flow:** Used for devices like Smart TVs.



### OpenID Connect (OIDC): The Identity Layer
OIDC sits on top of OAuth 2.0. While OAuth provides an **Access Token**, OIDC provides an **ID Token** (a JWT) that contains user profile information. This enables "Sign in with Google."

---

## 4. Access Control Patterns

### RBAC (Role-Based Access Control)
Users are assigned to roles, and roles are assigned permissions.
* *Admin Role:* Can `DELETE`, `CREATE`, `READ`.
* *User Role:* Can `READ`, `EDIT_OWN`.

### ABAC (Attribute-Based Access Control)
More granular than RBAC. Permissions are based on attributes (e.g., "Allow access only if the user is in the US and it is during business hours").

---

## 5. Backend Security Implementation Best Practices

### 1. Secure Password Storage
Never store passwords. Only store **Hashes**.
* **Salting:** Adding a random string to each password before hashing to prevent rainbow table attacks.
* **Algorithms:** Use `Argon2` (preferred) or `BCrypt`. Avoid `MD5` or `SHA-256` for passwords as they are too fast.

### 2. Defending Against Timing Attacks
Attackers can guess valid emails by measuring how long your server takes to respond. 
* **Fix:** Ensure your server takes the same amount of time to respond, regardless of whether the user exists or not.

### 3. Generic Error Handling
* **Bad:** "Email not found" or "Incorrect password."
* **Good:** "Invalid username or password."

### 4. Hybrid Token Strategy
To solve the JWT revocation problem, many companies use a **Short-lived Access Token** (5-15 mins) and a **Long-lived Refresh Token** (7-30 days) stored in a database.

---

## 6. Summary Table

| Feature | Stateful (Sessions) | Stateless (JWT) |
| :--- | :--- | :--- |
| **Storage** | Server-side (Redis/DB) | Client-side (Browser) |
| **Scalability** | Low (Centralized) | High (Decentralized) |
| **Revocation** | Instant | Difficult (Requires Blacklisting) |
| **Use Case** | Traditional Web Apps | Microservices & Mobile |

