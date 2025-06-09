# 🌐 2. API Development Fundamentals & HTTP Module

---

## 📘 1. What is an API?

An **API (Application Programming Interface)** is a set of functions or endpoints hosted on a server that can be accessed via a **client** (like browser, frontend app, Postman, etc.).

You send an **HTTP request** to a specific address (`URL`) using a particular method (`GET`, `POST`, etc.) and receive a **response**.

---

## 🔍 Components of an HTTP Request

| Component        | Description                                                |
| ---------------- | ---------------------------------------------------------- |
| **Base URL**     | IP or domain of the server (e.g., `http://128.198.17.8`)   |
| **Route**        | Path to a specific resource (e.g., `/home`, `/products`)   |
| **Method**       | HTTP method: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, etc. |
| **Headers**      | Metadata (e.g., `Content-Type`, `Authorization`)           |
| **Data/Body**    | Payload (usually in `POST`, `PUT`, `PATCH` methods)        |
| **Query Params** | Data in URL (e.g., `/delete?id=1`)                         |

---

## 📦 Components of an HTTP Response

| Component       | Description                                      |
| --------------- | ------------------------------------------------ |
| **Status Code** | Indicates result of request (e.g., `200`, `404`) |
| **Headers**     | Metadata sent back by server                     |
| **Body/Data**   | The actual data or message returned              |

---

## 🧭 Common HTTP Status Codes

- `200` – OK (Success)
- `201` – Created (Resource created)
- `400` – Bad Request
- `404` – Not Found
- `500` – Internal Server Error

---

## 🌐 Localhost and Ports

- `127.0.0.1` or `localhost`: Loopback IP (your own machine)
- `PORT`: Unique identifier for the server (e.g., `8000`, `3000`)

---

# 🛠️ 2. HTTP Module in Node.js

The `http` module in Node.js allows you to create a simple web server without using frameworks like Express.

---

## ✅ 2.1 Basic Server Creation

```js
const http = require("http");

http
  .createServer((req, res) => {
    res.end("This is my first response");
  })
  .listen(8000, () => {
    console.log("Server running on http://localhost:8000");
  });
```

---

## 🧭 2.2 Creating and Handling Routes

```js
const http = require("http");

http
  .createServer((req, res) => {
    if (req.url === "/home") {
      res.end("Welcome to Home Page");
    } else if (req.url === "/about") {
      res.end("My name is Nikhil");
    } else {
      res.end("404 - Page Not Found");
    }
  })
  .listen(8000);
```

---

## 📮 2.3 Handling HTTP Methods (GET, POST)

```js
const http = require("http");

http
  .createServer((req, res) => {
    if (req.url === "/home" && req.method === "GET") {
      res.end("GET: Home Page");
    } else if (req.url === "/about" && req.method === "POST") {
      res.end("POST: About Info");
    }
  })
  .listen(8000);
```

---

# 📤 3. Real GET API Example – Serving Data from File

```js
const http = require("http");
const fs = require("fs");

http
  .createServer((req, res) => {
    if (req.url === "/products" && req.method === "GET") {
      fs.readFile("./products.json", "utf-8", (err, data) => {
        res.setHeader("Content-Type", "application/json");
        if (err)
          return res.end(JSON.stringify({ message: "Error loading file" }));
        res.end(data);
      });
    }
  })
  .listen(8000);
```

---

## 🌐 Calling This API From Frontend (HTML/JS)

```js
const productUrl = "http://localhost:8000/products";
const show = document.querySelector(".show");

fetch(productUrl)
  .then((response) => {
    if (!response.ok) throw new Error("Failed to fetch");
    return response.json();
  })
  .then((data) => {
    show.innerHTML = data
      .map(
        (item) => `
      <div>
        <h2>${item.title}</h2>
        <p>${item.description || "No description"}</p>
      </div>`
      )
      .join("");
  })
  .catch((err) => {
    show.innerHTML = `<p>Error loading products: ${err.message}</p>`;
  });
```

---

# 📨 4. HTTP Methods Summary

| Method   | Purpose                |
| -------- | ---------------------- |
| `GET`    | Read / fetch data      |
| `POST`   | Create new data        |
| `PUT`    | Update existing data   |
| `PATCH`  | Partial update of data |
| `DELETE` | Remove data            |

---

## ❗ Sending POST Data (Overview – not implemented in raw HTTP)

In `http` module, you have to manually collect POST data like this:

```js
if (req.method === "POST") {
  let body = "";
  req.on("data", (chunk) => {
    body += chunk.toString();
  });

  req.on("end", () => {
    const parsedData = JSON.parse(body);
    console.log(parsedData);
    res.end("Data received");
  });
}
```

---

## 🔐 What is CORS?

**CORS (Cross-Origin Resource Sharing)** is a security mechanism that allows or blocks resources on a web page to be requested from another domain.

- For example, if your frontend runs on `http://localhost:3000` and backend on `http://localhost:8000`, the browser will block the request unless **CORS** is handled.
- To enable CORS in a Node.js server:

```js
res.setHeader("Access-Control-Allow-Origin", "*");
res.setHeader("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE");
res.setHeader("Access-Control-Allow-Headers", "Content-Type");
```

> For production apps, use the `cors` package if using Express.

---
