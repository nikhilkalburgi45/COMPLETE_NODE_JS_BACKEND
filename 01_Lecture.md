# ✅ 1. Node.js Fundamentals & `fs` Module

---

## 🔹 1.1 Function

A **function** is a reusable block of code designed to perform a specific task. Functions help in modularizing code and improving readability.

```js
function greet(name) {
  return `Hello, ${name}!`;
}

console.log(greet("Nikhil")); // Output: Hello, Nikhil!
```

---

## 🔹 1.2 Module

A **module** is simply a file that contains reusable code — usually a collection of related functions, objects, or classes.

- In Node.js, each file is treated as a separate module.
- You can use `module.exports` to expose functions or variables.

**Example:**

```js
// greetings.js
function sayHi(name) {
  return `Hi, ${name}`;
}

module.exports = { sayHi };
```

```js
// index.js
const greetings = require("./greetings");
console.log(greetings.sayHi("Nikhil"));
```

---

## 🔹 1.3 Package

A **package** (or **library**) is a collection of modules grouped together. Most packages are installed via **npm** (Node Package Manager).

Example packages:

- `express` - web framework
- `lodash` - utility functions
- `mongoose` - MongoDB ODM

---

# 📁 2. `fs` Module (File System)

The `fs` module in Node.js allows you to interact with the file system.
It is **pre-installed**, so you don’t need to install it manually.

---

## 📖 2.1 Reading Files

### ✅ Synchronous (Blocking)

```js
const fs = require("fs");

const data = fs.readFileSync("./file.txt", "utf-8");
console.log(data);
```

- `readFileSync()` blocks the execution until the file is read.
- Good for scripts or small tasks.

---

### ✅ Asynchronous (Non-blocking)

```js
const fs = require("fs");

fs.readFile("./file.txt", "utf-8", (err, data) => {
  if (err) return console.error(err);
  console.log(data);
});
```

- Doesn’t block the rest of your code.
- Recommended for performance in real-world apps.

---

## ✍️ 2.2 Writing Files

### ✅ Synchronous

```js
const fs = require("fs");

fs.writeFileSync("./output.txt", "Hello, World!");
```

- Overwrites existing content.

---

### ✅ Asynchronous

```js
fs.writeFile("./output.txt", "Hello, async world!", (err) => {
  if (err) console.error(err);
  else console.log("File written successfully");
});
```

---

## ➕ 2.3 Appending to Files

To avoid overwriting existing content:

```js
fs.appendFile("./output.txt", "\nAppended line", (err) => {
  if (err) console.error(err);
  else console.log("Content appended.");
});
```

---

## ❌ 2.4 Deleting Files

```js
fs.unlinkSync("./output.txt");
```

- `unlinkSync()` deletes the specified file synchronously.

---

## 🗃️ Extra (Check if File Exists)

```js
if (fs.existsSync("./file.txt")) {
  console.log("File exists.");
} else {
  console.log("File does not exist.");
}
```

---

# 🧠 3. `os` Module (Operating System)

The `os` module provides information about the operating system.

```js
const os = require("os");

console.log("Platform:", os.platform()); // e.g., 'win32', 'linux'
console.log("OS Name:", os.hostname()); // e.g., 'Nikhil-PC'
console.log("Free Memory:", os.freemem()); // in bytes
console.log("Total Memory:", os.totalmem()); // in bytes
console.log("CPU Info:", os.cpus()); // Array of CPU core info
console.log("User Info:", os.userInfo()); // Info about current user
```

---

# 🧩 Summary

| Concept   | Description                               |
| --------- | ----------------------------------------- |
| Function  | Reusable block of code                    |
| Module    | File containing related code              |
| Package   | Collection of modules                     |
| fs module | Read/write/delete files                   |
| os module | Access system-level info like memory, CPU |

---

Great! Let’s dive into **`req.cookies`** with detailed explanation, real-world example, and code.

---

# 8. `req.cookies`

### What is `req.cookies`?

- `req.cookies` holds **cookies sent by the client browser** in the HTTP request.

- Cookies are small pieces of data stored on the client side (browser) to keep track of user info like:

  - Authentication tokens
  - User preferences
  - Session IDs
  - Shopping cart contents

- To access cookies in Express, you usually use the **cookie-parser middleware** which parses cookies and adds them as a JavaScript object to `req.cookies`.

---

### Real-world use case:

Suppose you want to **check if a user is logged in** by looking for a cookie named `"authToken"`.

If the cookie exists and is valid, you allow access; otherwise, you redirect to login.

Cookies help persist login sessions or user-specific settings without hitting the database every time.

---

### How to set up and use `req.cookies`

---

### Step 1: Install cookie-parser

```bash
npm install cookie-parser
```

---

### Step 2: Use cookie-parser middleware in your Express app

```js
const express = require("express");
const cookieParser = require("cookie-parser");

const app = express();

app.use(cookieParser()); // this will parse cookies and populate req.cookies
```

---

### Step 3: Example of reading cookies

```js
app.get("/profile", (req, res) => {
  // Accessing the authToken cookie from the browser request
  const authToken = req.cookies.authToken;

  if (authToken) {
    // In real app, validate authToken, fetch user profile etc.
    res.send({ message: "Welcome back, user!", token: authToken });
  } else {
    res.status(401).send({ message: "Unauthorized, please login." });
  }
});
```

---

### Step 4: Example of setting cookies (for completeness)

```js
app.get("/login", (req, res) => {
  // After successful login, set an authToken cookie
  res.cookie("authToken", "123456abcdef", { httpOnly: true, maxAge: 3600000 }); // 1 hour expiry
  res.send({ message: "Logged in successfully, cookie set!" });
});
```

---

### Explanation:

- `req.cookies` contains an object of all cookies sent by the client.
- You access cookies by their key names: `req.cookies.cookieName`.
- Cookies can be set by the server using `res.cookie()` and the browser sends them back in subsequent requests.
- The option `httpOnly: true` makes cookies inaccessible to JavaScript on the client side (safer from XSS attacks).
- `maxAge` defines cookie expiry time.

---

### Why are cookies important?

- They maintain **stateful sessions** in the otherwise stateless HTTP protocol.
- Allow features like **remember me**, **shopping cart persistence**, and **user personalization**.

---
