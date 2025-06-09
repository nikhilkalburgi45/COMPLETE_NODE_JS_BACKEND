# 3. Connecting MongoDB to Node.js (with Mongoose)

---

## 📌 Overview

To connect MongoDB to Node.js using Mongoose, follow **3 fundamental steps**:

### 1️⃣ MongoDB Connection

### 2️⃣ Schema Creation

### 3️⃣ Model Creation

---

## 1️⃣ Connecting to MongoDB

### ✅ Using `async/await`:

```js
const mongoose = require("mongoose");

const connectDb = async () => {
  try {
    await mongoose.connect("mongodb://localhost:27017/practice");
    console.log("Database Connected Successfully");
  } catch (error) {
    console.log(error);
    console.log("Some Error While Connecting To Database");
  }
};

connectDb();
```

### ✅ Using Promises:

```js
const mongoose = require("mongoose");

const connectDb = () => {
  mongoose
    .connect("mongodb://localhost:27017/practice")
    .then(() => console.log("Database Connected Successfully"))
    .catch((error) => {
      console.log(error);
      console.log("Some Error While Connecting To Database");
    });
};

connectDb();
```

---

## ❓ What is Mongoose?

**Mongoose** is an **ODM (Object Data Modeling)** library for MongoDB and Node.js.
It provides a **schema-based solution** to model your application data and includes built-in validation, query building, and business logic hooks.

---

## 2️⃣ Schema Definition

```js
const mongoose = require("mongoose");

const userSchema = mongoose.Schema({
  name: {
    type: String,
    required: true,
  },
  email: {
    type: String,
    required: true,
  },
  password: {
    type: String,
    required: true,
  },
  age: {
    type: Number,
    required: true,
    min: [10, "Age Should Be Greater Than 10"],
    max: [90, "Age Should Be Smaller Than 90"],
  },
});
```

---

## 3️⃣ Creating a Model

```js
const userModel = mongoose.model("users", userSchema);
```

- **Schema**: Defines structure of documents
- **Model**: Provides an interface to interact with MongoDB collections

---

## ✅ CRUD Operations with Mongoose

### 🔼 1. Insert Data

```js
const userData = [
  {
    name: "nikhil",
    email: "nikhilkalburgi21@gmail.com",
    password: "12345678",
    age: 21,
  },
  { name: "nikita", email: "nikita@gmail.com", password: "12345678", age: 25 },
  { name: "rohit", email: "rohit@gmail.com", password: "12345678", age: 39 },
  { name: "virat", email: "virat@gmail.com", password: "12345678", age: 37 },
];

userModel
  .create(userData)
  .then((data) => console.log({ data, message: "Created/Added Successfully" }))
  .catch((err) => console.log(err));
```

---

### 🔽 2. Fetch All Data

```js
userModel
  .find()
  .then((data) => console.log({ data, message: "Fetched Data Successfully" }))
  .catch((err) => console.log("Error While Fetching", err));
```

---

### 🔍 3. Fetch Data with Conditions

```js
userModel
  .find({ age: { $gte: 35 } })
  .then((data) => console.log({ data, message: "Fetched Data Successfully" }))
  .catch((err) => console.log("Error While Fetching", err));
```

---

### ❌ 4. Delete Data

```js
userModel
  .deleteOne({ age: 37 })
  .then((info) => console.log({ info, message: "Data Deleted Successfully" }))
  .catch((err) => console.log("Error While Deleting", err));
```

---

### 🔁 5. Update Data

```js
userModel
  .updateOne({ name: "rohit" }, { $set: { age: 30, name: "sharma" } })
  .then((info) => console.log({ info, message: "Data Updated Successfully" }))
  .catch((err) => console.log("Error While Updating", err));
```

---

✅ **Bonus Terms**

- `create()` → Insert document(s)
- `find()` → Fetch all documents
- `findOne()` → Fetch single matching document
- `deleteOne()` / `deleteMany()` → Delete document(s)
- `updateOne()` / `updateMany()` → Update document(s)

---
