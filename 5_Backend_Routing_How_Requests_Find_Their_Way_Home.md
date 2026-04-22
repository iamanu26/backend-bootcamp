# 🛣️ Backend Routing: How Requests Find Their Way Home

In the world of HTTP, the **Method** (GET, POST, etc.) defines the **"What"** (your intent), but **Routing** defines the **"Where"** (the resource). Routing is the mechanism that maps a URL path to specific server-side logic (Handlers).

---

## 📍 The Routing Equation
A route on a server is a combination of the HTTP Method and the URL Path:
> **Method + Path = Unique Handler**

For example, `/api/books` can point to two entirely different sets of code depending on the intent:
* `GET /api/books` ➔ Triggers the `fetchBooks` function.
* `POST /api/books` ➔ Triggers the `createBook` function.

---

## 🏗️ Types of Routes

### 1. Static Routes 🧱
These are constant strings that never change. They always point to the same resource.
* **Example:** `GET /api/v1/status`
* **Use Case:** Health checks, about pages, or fetching a full list of items.

### 2. Dynamic Routes (Path Parameters) 🔄
These allow you to pass variables directly inside the URL path.
* **Syntax:** `/api/users/:id` (The colon `:` denotes a placeholder).
* **Example:** `GET /api/users/123`
* **Logic:** The server extracts `123`, treats it as the `id`, and queries the database for that specific user. 



### 3. Nested Routes 🪆
Used to express relationships between resources semantically. This is a core practice in RESTful APIs.
* **Example:** `GET /api/users/123/posts/456`
* **Semantic Meaning:** "Fetch post #456 belonging specifically to user #123."
* **Benefit:** It creates a human-readable "hierarchy" for your data.

---

## 🔍 Path Parameters vs. Query Parameters

| Feature | Path Parameters (`:id`) | Query Parameters (`?key=val`) |
| :--- | :--- | :--- |
| **Placement** | Inside the URL path. | After the `?` symbol. |
| **Role** | Identifies a **specific resource**. | **Filters, Sorts, or Paginates** data. |
| **Example** | `/products/shampoo-7` | `/products?category=hair&sort=price` |

**Why Query Params?** Since `GET` requests do not have a request body, query parameters are the standard way to send "search" or "filter" metadata to the server.

---

## 🛠️ Professional Routing Practices

### 🔄 Route Versioning
APIs evolve. To avoid breaking old mobile apps or frontend versions, engineers use versioning.
* **Pattern:** `/api/v1/products` ➔ `/api/v2/products`
* **Strategy:** You can deploy new features in `v2` while keeping `v1` active for legacy users. Once everyone migrates, you can **deprecate** `v1`.

### ⚠️ Catch-all (404) Routing
A "Wildcard" route (usually `*`) placed at the very end of your routing list.
* **Function:** If a request matches nothing else, it hits this route.
* **Purpose:** To return a clean, user-friendly JSON error (e.g., `{"error": "Route not found"}`) instead of a raw server crash or an empty response.



---

## 🎯 Summary
Routing provides the **semantic address** for your server's features. By combining static paths, dynamic parameters, and query strings, you can build a predictable and easy-to-use API for any frontend.

