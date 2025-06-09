# 4 Express.js

# 1. What is Express.js?

Express.js is a minimal and flexible Node.js web application framework that provides a robust set of features to build web and mobile applications. It simplifies the process of creating API endpoints and handling HTTP requests and responses by building on top of Node's native `http` module.

### Key points:

- Express is lightweight and unopinionated, giving you the freedom to structure your app as you like.
- It provides routing, middleware support, and HTTP utilities.
- Widely used for building REST APIs and web applications.

---

# 2. Creating a Simple Express Server

To start an Express app, you need to:

1. Import Express.
2. Create an app instance.
3. Define routes/endpoints.
4. Start the server by listening on a specific port.

Here’s a minimal example:

```js
const express = require("express"); // Import express module

const app = express(); // Create an Express app instance

const PORT = 8000; // Define the port number

app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});
```

This code creates a basic server listening on port 8000. You will add routes next to handle requests.

---

# 3. Handling GET Requests and Sending Responses

Express allows you to define routes to respond to HTTP methods like GET, POST, PUT, DELETE, etc.

Example of handling a simple GET request on the `/path` URL:

```js
app.get("/path", (req, res) => {
  res.send({ message: "GET Request Successful" });
});
```

- `app.get` listens for GET requests on `/path`.
- The callback function receives `req` (request) and `res` (response) objects.
- `res.send()` sends a response back to the client. It can send strings, JSON, buffers, etc.

---

# 4. Understanding Common `req` Properties

| Property      | Description                                   | Use Case                               |
| ------------- | --------------------------------------------- | -------------------------------------- |
| `req.body`    | Data sent in POST/PUT requests (usually JSON) | Creating or updating data              |
| `req.params`  | URL path variables                            | Fetch or operate on specific resources |
| `req.query`   | URL query strings (after `?`)                 | Filtering, sorting, pagination         |
| `req.headers` | HTTP headers sent by client                   | Authentication, content type handling  |
| `req.cookies` | Cookies sent by client                        | Session management, auth               |
| `req.method`  | HTTP method (GET, POST, etc.)                 | Logging, conditional logic             |
| `req.url`     | Full URL requested                            | Logging, debugging                     |
| `req.path`    | URL path only                                 | Route matching                         |

---

Sure, Nikhil! Let’s go **one by one**, with **detailed explanations** and **real-world examples** for each common `req` property in Express.js.

---

# 1. `req.body`

### What is `req.body`?

- Contains data sent by the client in the **request body**.
- Commonly used in **POST**, **PUT**, or **PATCH** requests.
- Data can be JSON, form data, or other formats.
- Requires middleware like `express.json()` to parse JSON data.

---

### Real-world use case:

Imagine you have a **user registration form** on your website where users enter their username, email, and password. When the user submits this form, your backend receives the data in `req.body`.

---

### Code example with explanation:

```js
const express = require("express");
const app = express();

// Middleware to parse JSON request bodies
app.use(express.json());

app.post("/register", (req, res) => {
  // Extract user data from request body
  const { username, email, password } = req.body;

  if (!username || !email || !password) {
    return res.status(400).send({ error: "All fields are required" });
  }

  // In a real app, save user to database here...

  res.status(201).send({
    message: "User registered successfully",
    user: { username, email },
  });
});

app.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

---

### How to test:

Send a POST request to `http://localhost:3000/register` with this JSON body:

```json
{
  "username": "nikhil123",
  "email": "nikhil@example.com",
  "password": "securepass"
}
```

---

### Explanation:

- `express.json()` middleware parses incoming JSON data.
- `req.body` holds the parsed JSON object.
- We validate required fields, then proceed to use data (like saving to DB).
- Respond with a confirmation message and some user info.

---

Great! Let’s move on to the next:

---

# 2. `req.params`

### What is `req.params`?

- Holds **dynamic URL segments** (route parameters).
- Used to capture values embedded in the URL path.
- Commonly used for accessing specific resources by ID, name, or other unique identifiers.

---

### Real-world use case:

Imagine an **e-commerce site** where you want to fetch details of a product by its unique `productId` embedded in the URL.

---

### Code example with explanation:

```js
const express = require("express");
const app = express();

// Example product database (array of objects)
const products = [
  { id: "101", name: "Laptop", price: 1500 },
  { id: "102", name: "Smartphone", price: 700 },
  { id: "103", name: "Headphones", price: 100 },
];

app.get("/products/:productId", (req, res) => {
  // Extract productId from URL params
  const productId = req.params.productId;

  // Find product by id
  const product = products.find((p) => p.id === productId);

  if (!product) {
    return res.status(404).send({ error: "Product not found" });
  }

  // Send product details
  res.send(product);
});

app.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

---

### How to test:

Visit `http://localhost:3000/products/102` in the browser or use a tool like Postman. It will return:

```json
{
  "id": "102",
  "name": "Smartphone",
  "price": 700
}
```

---

### Explanation:

- The URL path has a **dynamic parameter** `:productId`.
- Express extracts this into `req.params.productId`.
- We use this ID to find the product in the sample database.
- If found, we send the product data, otherwise send a 404 error.

---

Awesome! Let’s dive into:

---

# 3. `req.query`

### What is `req.query`?

- Holds **query string parameters** from the URL.
- These are the key-value pairs after the `?` in a URL.
- Commonly used for **filtering, searching, sorting, pagination**, etc.

---

### Real-world use case:

Imagine an **online bookstore** where users can filter books by genre, author, or sort them by price.

---

### Code example with explanation:

```js
const express = require("express");
const app = express();

// Sample books database
const books = [
  { id: "1", title: "Book A", author: "Author1", genre: "fiction", price: 20 },
  {
    id: "2",
    title: "Book B",
    author: "Author2",
    genre: "non-fiction",
    price: 15,
  },
  { id: "3", title: "Book C", author: "Author1", genre: "fiction", price: 25 },
  { id: "4", title: "Book D", author: "Author3", genre: "history", price: 30 },
];

app.get("/books", (req, res) => {
  // Extract query parameters
  const { genre, author, sortByPrice } = req.query;

  let filteredBooks = [...books];

  // Filter by genre if provided
  if (genre) {
    filteredBooks = filteredBooks.filter(
      (book) => book.genre.toLowerCase() === genre.toLowerCase()
    );
  }

  // Filter by author if provided
  if (author) {
    filteredBooks = filteredBooks.filter(
      (book) => book.author.toLowerCase() === author.toLowerCase()
    );
  }

  // Sort by price if sortByPrice is 'asc' or 'desc'
  if (sortByPrice === "asc") {
    filteredBooks.sort((a, b) => a.price - b.price);
  } else if (sortByPrice === "desc") {
    filteredBooks.sort((a, b) => b.price - a.price);
  }

  res.send(filteredBooks);
});

app.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

---

### How to test:

Try visiting these URLs in browser or Postman:

- `http://localhost:3000/books?genre=fiction`
  Returns books with genre "fiction".

- `http://localhost:3000/books?author=Author1&sortByPrice=asc`
  Returns books by Author1 sorted by price ascending.

- `http://localhost:3000/books`
  Returns all books.

---

### Explanation:

- `req.query` extracts query parameters from the URL.
- You can send multiple filters or sort options dynamically.
- The server filters and sorts the data accordingly.
- Useful for endpoints where users need to search or filter large data sets.

---

Great! Let’s talk about:

---

# 4. `req.headers`

### What is `req.headers`?

- Contains the **HTTP headers** sent by the client in the request.
- Headers carry **metadata** like content type, authorization tokens, client info, cookies, etc.
- Used for things like **authentication, content negotiation, caching, user agent info**, and more.

---

### Real-world use case:

Imagine an API endpoint that requires a **JWT token in the Authorization header** to verify if the user is logged in before allowing access.

---

### Code example with explanation:

```js
const express = require("express");
const app = express();

// Middleware to check Authorization header
function authenticate(req, res, next) {
  const authHeader = req.headers["authorization"]; // or req.get('Authorization')

  if (!authHeader) {
    return res.status(401).send({ message: "Authorization header missing" });
  }

  // Normally, the token is in format "Bearer <token>"
  const token = authHeader.split(" ")[1];
  if (!token) {
    return res
      .status(401)
      .send({ message: "Token missing in Authorization header" });
  }

  // For demo: Just check if token equals 'secret123'
  if (token !== "secret123") {
    return res.status(403).send({ message: "Invalid token" });
  }

  // Token is valid, proceed
  next();
}

// Protected route example
app.get("/profile", authenticate, (req, res) => {
  res.send({ message: "This is the protected user profile data" });
});

app.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

---

### How to test:

Make a GET request to `http://localhost:3000/profile` with the header:

```
Authorization: Bearer secret123
```

- If the header is missing or token is invalid, you get an error.
- If token is correct, you get the protected data.

---

### Explanation:

- `req.headers` gives you access to all headers, but usually, you access headers case-insensitively by name.
- Authentication tokens like JWTs are typically sent in the `Authorization` header.
- Middleware intercepts the request, extracts the token from headers, verifies it, then allows or denies access.
- This pattern secures APIs by ensuring only authorized users can access certain routes.

---

Sure! Let’s dive into:

---

# 5. `req.path`

### What is `req.path`?

- `req.path` gives you the **URL path part** of the incoming request — basically the route without the query string or protocol/host.
- For example, if the full URL is `http://example.com/products/123?sort=asc`, then:

  - `req.path` will be `/products/123`
  - `req.url` would be `/products/123?sort=asc`

---

### Real-world use case:

You want to **log or track which API routes are being accessed** on your server for monitoring or analytics.

Or you want to build middleware that **blocks certain paths** temporarily for maintenance.

---

### Code example with explanation:

```js
const express = require("express");
const app = express();

// Middleware to log requested path
app.use((req, res, next) => {
  console.log(`Requested Path: ${req.path}`);
  next();
});

// Example routes
app.get("/products/:id", (req, res) => {
  res.send({ message: `Product ID requested: ${req.params.id}` });
});

app.get("/about", (req, res) => {
  res.send({ message: "About page" });
});

app.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

---

### How to test:

- Request: `GET http://localhost:3000/products/123?sort=asc`

  - Console will log: `Requested Path: /products/123`

- Request: `GET http://localhost:3000/about`

  - Console will log: `Requested Path: /about`

---

### Explanation:

- `req.path` extracts the path part only, ignoring query parameters.
- Useful for logging or conditional handling based on the path.
- In middleware, you can use `req.path` to allow/block routes dynamically.

---

Got it! Let’s explore:

---

# 6. `req.url`

### What is `req.url`?

- `req.url` contains the **full URL part after the domain**, including the **path + query string**.
- So for `http://example.com/products/123?sort=asc&limit=10`:

  - `req.url` will be `/products/123?sort=asc&limit=10`

- This differs from `req.path` which excludes query strings.

---

### Real-world use case:

Imagine you want to **log full requested URL for debugging** or **track users' filter/sort parameters** in analytics.

Or you want to **redirect users based on specific query parameters**.

---

### Code example with explanation:

```js
const express = require("express");
const app = express();

// Middleware to log full requested URL (path + query string)
app.use((req, res, next) => {
  console.log(`Full URL requested: ${req.url}`);
  next();
});

// Example route
app.get("/search", (req, res) => {
  res.send({ message: "Search endpoint hit" });
});

app.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

---

### How to test:

- Request: `GET http://localhost:3000/search?q=phone&category=electronics`

  - Console logs: `Full URL requested: /search?q=phone&category=electronics`

---

### Explanation:

- `req.url` gives the exact URL path plus query parameters as the client requested.
- Useful for precise logging or middleware decisions where query params matter.

Great! Let’s go step-by-step with explanations and real-world context for each part of your code. I’ll explain each snippet, its purpose, and how it fits in a real app scenario. Ready? Here we go:

---

Awesome! Let’s dive into **Middleware** next.

---

### 1. How to send a response and handle GET requests

```js
app.get("/path", (req, res) => {
  res.send({ Message: "Get Request Success" });
});
```

**Explanation:**

- `app.get` defines an endpoint that responds to HTTP GET requests to `/path`.
- `req` is the incoming request object.
- `res` is the response object you use to send back data.
- `res.send()` sends the response back to the client.

**Real-world example:**
Imagine you have a website where users can check the current status of a service. When they hit `/status`, you send back a JSON message confirming everything is OK.

```js
app.get("/status", (req, res) => {
  res.send({ status: "Server is running smoothly" });
});
```

This way, a front-end or a monitoring tool can call `/status` to check if your backend is alive.

Great! Let’s move on to **Handling POST Requests**.

---

## 3. Handling POST Requests in Express.js

---

### What is a POST request?

A **POST request** is typically used to send data from the client (like a form or JSON payload) to the server to create or update resources.

---

### How to handle POST request data?

In Express, POST data is sent in the **body** of the request. To access this data, you need to parse it — usually as JSON.

---

### Real-world example:

Imagine you have a website where users can submit their details through a form. The server needs to receive and process this user data.

---

### Example Code:

```js
// Middleware to parse incoming JSON data from the request body
app.use(express.json());

app.post("/users", (req, res) => {
  console.log("Received User Data:", req.body);

  // You can save this data to database or process it further
  res.send({ message: "User data received successfully", data: req.body });
});
```

---

### Explanation:

- `app.use(express.json())` is middleware that parses incoming JSON body and converts it to a JavaScript object you can access via `req.body`.
- In the route `/users`, we receive the POST request.
- `req.body` contains the JSON sent by the client.
- We log it and send back a confirmation response.

---

### How to test this?

Use tools like **Postman** or **curl**:

```bash
curl -X POST http://localhost:8000/users -H "Content-Type: application/json" -d '{"name":"Nikhil","age":21}'
```

This sends JSON data `{ "name": "Nikhil", "age": 21 }` to your POST endpoint.

---

### Summary:

- Use `express.json()` middleware to parse JSON POST bodies.
- Access the data via `req.body`.
- Respond with success or error messages.

---

Awesome! Let’s move on to **Handling DELETE Requests** in Express.js.

---

## 4. Handling DELETE Requests

---

### What is a DELETE request?

A **DELETE request** is used to delete a resource from the server, typically identified by some parameter like an `id`.

---

### Real-world example:

Suppose you have a user management system, and you want to delete a user by their unique user ID.

---

### Example Code:

```js
app.delete("/users/:id", (req, res) => {
  const userId = req.params.id;
  console.log("User ID to delete:", userId);

  // Here you would normally delete the user from the database using userId
  // For example: UserModel.deleteOne({ _id: userId })

  res.send({ message: `User with ID ${userId} deleted successfully.` });
});
```

---

### Explanation:

- The route uses `:id` as a **URL parameter** to capture the user ID from the request URL.
- You access this parameter via `req.params.id`.
- You can then delete the user from your database using this ID (example shows a placeholder comment for this).
- After deletion, send back a confirmation message to the client.

---

### How to test this?

Using **curl**:

```bash
curl -X DELETE http://localhost:8000/users/123
```

Here, `123` is the user ID you want to delete.

---

### Summary:

- Use `app.delete()` for DELETE requests.
- Use URL params (`:id`) to specify what resource to delete.
- Access params with `req.params`.
- Perform deletion in the database and respond accordingly.

---

Great! Let’s dive into the next important HTTP method:

---

## 5. Handling PUT Requests (Update)

---

### What is a PUT request?

A **PUT request** is used to update an existing resource on the server. You typically send:

- The **resource identifier** (like an ID) in the URL as a parameter.
- The **updated data** in the request body.

---

### Real-world example:

Let’s say you want to update user details (like name or age) by their user ID.

---

### Example Code:

```js
app.put("/users/:id", (req, res) => {
  const userId = req.params.id; // Get user ID from URL
  const updatedData = req.body; // Get updated data from request body

  console.log("User ID to update:", userId);
  console.log("Updated data:", updatedData);

  // Normally, update the user in your database:
  // UserModel.updateOne({ _id: userId }, updatedData)

  res.send({ message: `User with ID ${userId} updated successfully.` });
});
```

---

### Explanation:

- The route uses `:id` as URL param to identify which user to update.
- Updated data is sent in the **request body** and accessed via `req.body`.
- You would call your database’s update method to update the user with the new data.
- After updating, respond with a success message.

---

### Important: Enable JSON parsing middleware

To access `req.body` in Express for JSON data, make sure you have:

```js
app.use(express.json());
```

---

### How to test this?

Using **curl**:

```bash
curl -X PUT http://localhost:8000/users/123 \
  -H "Content-Type: application/json" \
  -d '{"name": "Nikhil", "age": 25}'
```

This updates user with ID `123` to have the name "Nikhil" and age 25.

---

### Summary:

- Use `app.put()` to handle updates.
- Get the resource ID from URL params via `req.params`.
- Get update data from `req.body`.
- Perform DB update and send back a response.

---

### 2. Middleware — What is it and how to use it?

Middleware is basically a function that sits between the client request and the endpoint handler. It can inspect, modify, or block the request before it reaches the actual route.

---

#### Real-world scenario:

Imagine you have a website where some pages are restricted based on user age. You want to block users younger than 18 from accessing certain routes.

---

#### Example:

```js
app.get("/users/:age", middleMan, (req, res) => {
  res.send({ message: "Access granted, welcome!" });
});

function middleMan(req, res, next) {
  if (req.params.age <= 18) {
    res.send({ message: "Access denied: Age should be greater than 18" });
  } else {
    next(); // Pass control to the next middleware or route handler
  }
}
```

---

**Explanation:**

- The middleware function `middleMan` checks the age param in the URL.
- If the user is under 18, it stops the request by sending a response immediately.
- If the user is 19 or above, it calls `next()` to allow the request to continue to the actual endpoint.

---

#### Problem:

If you have 200 routes where this check applies, you wouldn’t want to add this middleware to every single route manually, right?

**Solution:**

You can use `app.use()` to apply the middleware globally, like this:

```js
app.use(middleMan);
```

Now, `middleMan` will run for **every** incoming request before reaching any route.

---

#### Real-world use of middleware globally:

- Authentication checks (Is the user logged in?)
- Logging every request
- CORS policy enforcement
- Input validation

---
