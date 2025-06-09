# 5 Authentication and Authorization

## 1. Authentication

**Definition:**
Authentication is the process of verifying the identity of a user or system. It answers the question: **"Who are you?"**

- When a user logs in by providing credentials (like username and password), the system checks if those credentials are valid.
- If valid, the user is considered authenticated.

**Example:** Logging in to your app with an email and password.

---

## 2. Authorization

**Definition:**
Authorization is the process of determining what an authenticated user is allowed to do. It answers the question: **"What can you do?"**

- After authentication, the system checks the user's permissions or roles.
- For example, only admins can access the admin dashboard; normal users cannot.

---

## 3. JSON Web Token (JWT)

**What is JWT?**
JWT is a compact, URL-safe means of representing claims to be transferred between two parties. It is commonly used to securely transmit user identity information between client and server after authentication.

- JWT acts like a "proof of identity" token.
- After login, the server generates a JWT and sends it to the client.
- The client sends the JWT with each request to access protected resources.
- The server verifies the JWT to allow or deny access.

---

## 4. How JWT Works - Structure of a JWT

A JWT consists of three parts, separated by dots (`.`):

```
header.payload.signature
```

1. **Header**
   Contains metadata about the token, usually the type of token (JWT) and the signing algorithm (e.g., HS256).

2. **Payload**
   Contains the claims — data you want to include in the token (e.g., user ID, email). This is the information the server can trust once verified.

3. **Signature**
   Created by signing the header and payload with a secret key using the specified algorithm. This ensures the token's integrity and authenticity.

---

## 5. When and How to Create a JWT Token

- You create a JWT token **after** successful user login or registration.
- The server signs the token with a secret key and sends it back to the client.
- The client stores this token (usually in local storage or cookies).
- The client includes the token in the `Authorization` header of future requests to protected routes.

**Example code to create JWT token:**

```js
const jwt = require("jsonwebtoken");

// After successful login
const userPayload = { id: user.id, email: user.email }; // unique user data

jwt.sign(userPayload, "your_secret_key", { expiresIn: "1h" }, (err, token) => {
  if (err) {
    return res.status(500).send({ message: "Error generating token" });
  }
  res.send({ token });
});
```

**Notes:**

- Use a strong, secure secret key (keep it safe, do not expose publicly).
- You can specify token expiration (e.g., 1 hour) for security.

---

# 6. How to Secure Any Endpoint Using JWT Middleware

When you have protected routes, you want only authenticated users with valid JWT tokens to access them.

---

### Step 1: Create a Middleware Function to Verify JWT Token

This middleware will:

- Check if the request contains an Authorization header with a Bearer token.
- Extract the token.
- Verify the token using the secret key.
- If valid, allow the request to continue.
- If invalid or missing, respond with an error.

---

### Improved JWT Verification Middleware

```js
const jwt = require("jsonwebtoken");

const verifyToken = (req, res, next) => {
  try {
    // Get the Authorization header value
    const authHeader = req.headers.authorization;

    // Check if header is present and starts with 'Bearer '
    if (!authHeader || !authHeader.startsWith("Bearer ")) {
      return res
        .status(401)
        .json({ message: "Authorization token missing or malformed" });
    }

    // Extract the token string from header
    const token = authHeader.split(" ")[1];

    // Verify the token
    jwt.verify(token, "your_secret_key", (err, decoded) => {
      if (err) {
        return res
          .status(403)
          .json({ message: "Invalid or expired token. Please login again." });
      }

      // Attach the decoded user info to the request object for use in next middleware/routes
      req.user = decoded;

      // Proceed to next middleware or route handler
      next();
    });
  } catch (error) {
    res.status(500).json({ message: "Internal server error" });
  }
};
```

---

### Explanation:

- `req.headers.authorization` contains the token usually in format:
  `"Bearer <token>"`.
- We check if it exists and is correctly formatted.
- `jwt.verify` checks the token validity with the secret key.
- If valid, `decoded` contains the payload (user info), which you can save in `req.user`.
- If not valid or expired, return 403 Forbidden.
- Catch other errors gracefully.

---

### Step 2: Use Middleware to Protect Routes

You can use the `verifyToken` middleware to protect any route:

```js
app.get("/testing", verifyToken, (req, res) => {
  res.json({
    message: "Confidential data accessible only with a valid token.",
    user: req.user, // this contains the decoded user info from JWT
  });
});
```

---

### Summary So Far:

- Middleware extracts JWT from headers.
- Verifies token.
- Allows access if token is valid.
- Blocks access if token missing or invalid.

---

# 7. Implementing User Login with JWT Token Generation

### Typical login flow:

1. User sends login request with credentials (e.g., email and password).
2. Server verifies the credentials against the database.
3. If valid, server creates a JWT token with user info.
4. Server sends the JWT token back to the client.
5. Client stores the token and sends it in Authorization header for protected requests.

---

### Sample Express.js Login Route

```js
const express = require("express");
const jwt = require("jsonwebtoken");
const bcrypt = require("bcrypt"); // for password hashing comparison

const app = express();
app.use(express.json());

// Simulated user data - in real app, fetch from database
const users = [
  { id: 1, email: "user@example.com", passwordHash: "$2b$10$somethinghashed" },
];

app.post("/login", async (req, res) => {
  const { email, password } = req.body;

  // 1. Find user by email
  const user = users.find((u) => u.email === email);
  if (!user) {
    return res.status(401).json({ message: "Invalid email or password" });
  }

  // 2. Compare passwords (hashed)
  const passwordMatch = await bcrypt.compare(password, user.passwordHash);
  if (!passwordMatch) {
    return res.status(401).json({ message: "Invalid email or password" });
  }

  // 3. Create JWT token payload (minimal info)
  const payload = { id: user.id, email: user.email };

  // 4. Sign token with secret key and expiration
  jwt.sign(payload, "your_secret_key", { expiresIn: "1h" }, (err, token) => {
    if (err) {
      return res.status(500).json({ message: "Error generating token" });
    }
    // 5. Send token to client
    res.json({ token });
  });
});
```

---

# 8. Token Expiration and Refresh Tokens

### Why Token Expiration?

- JWT tokens are **stateless**: once issued, they cannot be revoked until they expire.
- To enhance security, tokens are given a short lifetime (e.g., 1 hour).
- After expiration, the client needs to get a new token to continue using protected endpoints.

---

### Refresh Tokens

- To avoid asking users to login frequently, apps use **refresh tokens**.
- A refresh token is a long-lived token issued alongside the short-lived JWT access token.
- When access token expires, client sends refresh token to get a new access token without re-login.
- Refresh tokens are stored securely (often HTTP-only cookies) and must be protected.

**Note:** Implementing refresh tokens involves more complexity: storing refresh tokens safely, revoking them, etc.

---

# 9. Logout and Token Invalidation

- Since JWT is stateless, "logging out" on server side is tricky because tokens can't be deleted.
- Common approaches:

  - **Client-side logout:** Client deletes token from storage (localStorage, cookies), so it stops sending it.
  - **Token blacklist:** Server keeps a list of revoked tokens and checks against it (can affect performance).
  - **Short token lifetime + refresh tokens:** Limits window of token misuse.

---

# 10. Refresh Token Implementation — Basic Example

### Key points:

- When the user logs in, server generates **two tokens**:

  - **Access Token:** Short-lived (e.g., 15 minutes to 1 hour), used to access protected resources.
  - **Refresh Token:** Long-lived (e.g., days or weeks), used to get a new access token when the current one expires.

- Refresh tokens are usually stored in an **HTTP-only cookie** for security.
- Server validates refresh tokens and issues new access tokens.
- On logout, refresh tokens can be invalidated or deleted.

---

### Step 1: Login Route Generates Both Tokens

```js
const express = require("express");
const jwt = require("jsonwebtoken");
const bcrypt = require("bcrypt");
const app = express();
app.use(express.json());

// Simulated user data for demo
const users = [
  { id: 1, email: "user@example.com", passwordHash: "$2b$10$somethinghashed" },
];

// In-memory store for refresh tokens (for demo only, use DB in production)
let refreshTokens = [];

app.post("/login", async (req, res) => {
  const { email, password } = req.body;

  const user = users.find((u) => u.email === email);
  if (!user)
    return res.status(401).json({ message: "Invalid email or password" });

  const passwordMatch = await bcrypt.compare(password, user.passwordHash);
  if (!passwordMatch)
    return res.status(401).json({ message: "Invalid email or password" });

  const payload = { id: user.id, email: user.email };

  // Create access token (short lived)
  const accessToken = jwt.sign(payload, "access_secret_key", {
    expiresIn: "15m",
  });

  // Create refresh token (long lived)
  const refreshToken = jwt.sign(payload, "refresh_secret_key", {
    expiresIn: "7d",
  });

  // Store refresh token (in DB or memory)
  refreshTokens.push(refreshToken);

  // Send tokens to client
  res.json({
    accessToken,
    refreshToken,
  });
});
```

---

### Step 2: Refresh Token Route to Get New Access Token

```js
app.post("/token", (req, res) => {
  const { token } = req.body; // client sends refresh token in request body

  if (!token)
    return res.status(401).json({ message: "Refresh Token required" });
  if (!refreshTokens.includes(token)) {
    return res.status(403).json({ message: "Invalid Refresh Token" });
  }

  jwt.verify(token, "refresh_secret_key", (err, user) => {
    if (err) return res.status(403).json({ message: "Invalid Refresh Token" });

    const payload = { id: user.id, email: user.email };

    // Issue new access token
    const accessToken = jwt.sign(payload, "access_secret_key", {
      expiresIn: "15m",
    });

    res.json({ accessToken });
  });
});
```

---

### Step 3: Logout Route to Delete Refresh Token

```js
app.post("/logout", (req, res) => {
  const { token } = req.body;
  // Remove refresh token from store
  refreshTokens = refreshTokens.filter((t) => t !== token);
  res.json({ message: "Logged out successfully" });
});
```

---

### Step 4: Protect Routes Using `verifyToken` Middleware with Access Token

Use the same `verifyToken` middleware from earlier, but verify with the access token secret:

```js
const verifyToken = (req, res, next) => {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith("Bearer ")) {
    return res
      .status(401)
      .json({ message: "Authorization token missing or malformed" });
  }

  const token = authHeader.split(" ")[1];

  jwt.verify(token, "access_secret_key", (err, user) => {
    if (err) {
      return res.status(403).json({ message: "Invalid or expired token" });
    }

    req.user = user;
    next();
  });
};
```

---

### Summary Flow:

- User logs in → gets **access token** + **refresh token**.
- Client sends access token to access protected endpoints.
- When access token expires → client sends refresh token to `/token` to get new access token.
- User logs out → refresh token is invalidated on the server.
- Access token eventually expires and cannot be refreshed without a valid refresh token.

---
