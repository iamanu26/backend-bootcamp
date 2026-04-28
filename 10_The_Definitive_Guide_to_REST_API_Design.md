# 🚀 The Definitive Guide to REST API Design

> A masterclass in designing intuitive, scalable, and standardized API interfaces. This guide bridges the gap between theoretical REST constraints and modern industry best practices.

---

## Table of Contents

1. [The Origins: Roy Fielding's REST](#1-the-origins-roy-fieldings-rest)
2. [Resource-Oriented Naming & Path Design](#2-resource-oriented-naming--path-design)
3. [HTTP Methods & Idempotency](#3-http-methods--idempotency)
4. [Advanced Data Patterns](#4-advanced-data-patterns)
5. [Status Codes: The Server's Language](#5-status-codes-the-servers-language)
6. [Best Practices for Professional APIs](#6-best-practices-for-professional-apis)

---

## 1. The Origins: Roy Fielding's REST

**REST** (Representational State Transfer) was introduced in **2000 by Roy Fielding**. It is not a protocol (like HTTP) but an **architectural style** designed to make the web more scalable and independent.

### The 6 Architectural Constraints

| # | Constraint | Description |
|---|-----------|-------------|
| 1 | **Client-Server** | Decouples the UI from backend data storage, allowing both to evolve independently. |
| 2 | **Statelessness** | The server stores no session state. Every request must be self-descriptive — containing all info needed to process it. |
| 3 | **Uniform Interface** | All resources must be accessed via a consistent method (URIs + HTTP Verbs). |
| 4 | **Layered System** | The client cannot tell if it is connected directly to the end server or an intermediate (Load Balancer / CDN / Proxy). |
| 5 | **Cacheability** | Responses must define themselves as cacheable to prevent redundant processing and reduce server load. |
| 6 | **Code on Demand** *(Optional)* | Servers can temporarily extend client functionality by transferring executable code (e.g., JavaScript). |

---

## 2. Resource-Oriented Naming & Path Design

In REST, everything is a **Resource**. The URL represents the **Noun**, and the HTTP Method represents the **Verb**.

### Naming Rules

**✅ Plurals are Mandatory**
Use `/users`, `/orders`, `/products`. Even when accessing a single item, keep the resource name plural.

```
GET /users/42        ✅ Correct
GET /user/42         ❌ Wrong
```

**✅ Hierarchical Paths**
Use the path to express relationships between resources.

```
GET /organizations/5/projects/12
# Project 12 belongs to Organization 5
```

**✅ Kebab-case URLs**
Use hyphens for multi-word resource names. Avoid underscores or CamelCase.

```
GET /user-profiles    ✅ Correct
GET /userProfiles     ❌ Wrong
GET /user_profiles    ❌ Wrong
```

**✅ No Actions in URLs**
The HTTP method conveys the action. The URL should only describe the resource.

```
DELETE /users/42        ✅ Correct
GET    /getAllUsers      ❌ Wrong
POST   /createOrder     ❌ Wrong
```

---

## 3. HTTP Methods & Idempotency

> An operation is **Idempotent** if performing it multiple times has the **same effect** as performing it once.

| Method | Role | Idempotent | Notes |
|--------|------|:----------:|-------|
| `GET` | Read / Fetch | ✅ Yes | No changes to DB. Safe for repeated calls. |
| `POST` | Create / Action | ❌ No | Creates a new record every time. Also used for non-CRUD "actions". |
| `PUT` | Replace | ✅ Yes | Replaces the **entire** resource with the new payload. |
| `PATCH` | Partial Update | ✅ Yes | Modifies **only specific fields** provided in the payload. |
| `DELETE` | Remove | ✅ Yes | Resource is gone. 1st call deletes; 2nd call returns 404 — but state remains the same. |

### PUT vs PATCH — What's the Difference?

```json
// Original resource
{ "id": 1, "name": "Alice", "role": "admin", "active": true }

// PUT /users/1  →  Replaces the entire resource (missing fields are removed/nulled)
{ "name": "Alice Updated" }
// Result: { "id": 1, "name": "Alice Updated" }  ← role and active are GONE

// PATCH /users/1  →  Updates only the fields you send
{ "name": "Alice Updated" }
// Result: { "id": 1, "name": "Alice Updated", "role": "admin", "active": true }  ← rest preserved
```

---

## 4. Advanced Data Patterns

### A. Pagination — The "Portion" Pattern

Never return a full database table in one request. It kills performance and overwhelms the client.

**Request:**
```
GET /tasks?page=1&limit=20
```

**Response:**
```json
{
  "data": [ /* array of 20 task objects */ ],
  "metadata": {
    "totalCount": 450,
    "page": 1,
    "limit": 20,
    "totalPages": 23
  }
}
```

> 💡 **Tip:** Always return `metadata` alongside `data`. This tells clients how to build pagination controls without extra requests.

---

### B. Filtering & Sorting

Use query parameters to let clients customize their view **without creating new endpoints**.

```
# Filtering
GET /tasks?status=completed&priority=high

# Sorting
GET /tasks?sortBy=createdAt&order=desc

# Combined
GET /tasks?status=open&sortBy=dueDate&order=asc&page=1&limit=10
```

---

### C. Custom Actions

For non-CRUD business logic (e.g., "Archive", "Deploy", "Clone"), append the action after the resource ID using a `POST` request.

```
POST /projects/101/archive
POST /servers/5/reboot
POST /invoices/88/send
POST /users/42/reset-password
```

> This pattern keeps your URLs noun-based while still supporting rich business workflows.

---

## 5. Status Codes: The Server's Language

Using the correct HTTP status code is vital for a great **Developer Experience (DX)**.

### 2xx — Success

| Code | Name | When to Use |
|------|------|-------------|
| `200` | OK | General success for `GET`, `PUT`, `PATCH`. |
| `201` | Created | Successful `POST` that creates a new resource. |
| `204` | No Content | Successful `DELETE` — nothing to return. |

### 4xx — Client Errors

| Code | Name | When to Use |
|------|------|-------------|
| `400` | Bad Request | Validation failed — it's the client's fault. |
| `401` | Unauthorized | Missing or invalid authentication token. |
| `403` | Forbidden | Authenticated, but lacks the required permissions (RBAC). |
| `404` | Not Found | Resource ID does not exist. Note: an **empty list** should return `200 []`, not `404`. |
| `429` | Too Many Requests | Rate limiting has been triggered. |

### 5xx — Server Errors

| Code | Name | When to Use |
|------|------|-------------|
| `500` | Internal Server Error | The global catch-all for unexpected backend crashes. |

### Common Mistake: 404 vs 200 for Empty Results

```json
// ❌ Wrong — returning 404 for an empty collection
GET /users?role=superadmin
// Status: 404

// ✅ Correct — the endpoint exists; the collection is just empty
GET /users?role=superadmin
// Status: 200
// Body: { "data": [], "metadata": { "totalCount": 0 } }
```

---

## 6. Best Practices for Professional APIs

### Sane Defaults
If the client doesn't provide parameters, use sensible fallbacks:
- Default `limit` → `20`
- Default `sort` → newest first (`createdAt DESC`)

### No Abbreviations
Clarity always beats brevity.

```
✅  organizationId
❌  org_id

✅  totalCount
❌  cnt

✅  isActive
❌  flag
```

### CamelCase JSON Keys
Always use `camelCase` for response keys to match standard JavaScript/frontend expectations.

```json
{
  "userId": 42,
  "firstName": "Alice",
  "createdAt": "2025-01-15T10:30:00Z"
}
```

### Version Your API
Always prefix routes with `/v1/`. This lets you release `/v2/` later without breaking existing clients.

```
/v1/users
/v1/orders
/v2/users  ← breaking change isolated here
```

### Write Interactive Documentation
Use **Swagger (OpenAPI)**. A backend without documentation is like a library without a catalog.

```yaml
# openapi.yaml example snippet
openapi: 3.0.0
info:
  title: My API
  version: 1.0.0
paths:
  /users/{id}:
    get:
      summary: Get a user by ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
```

---

## Quick Reference Cheat Sheet

```
RESOURCE NAMING
  ✅ /users          Plural nouns
  ✅ /users/42       Access single resource (still plural)
  ✅ /users/42/posts Nested relationships
  ✅ /user-profiles  Kebab-case

HTTP METHODS
  GET    /users        → List all users
  POST   /users        → Create a new user
  GET    /users/42     → Get user #42
  PUT    /users/42     → Replace user #42 entirely
  PATCH  /users/42     → Update specific fields of user #42
  DELETE /users/42     → Delete user #42

QUERY PARAMS
  ?page=1&limit=20           Pagination
  ?status=active             Filtering
  ?sortBy=createdAt&order=desc  Sorting

STATUS CODES
  200 OK           →  Read / Update success
  201 Created      →  POST success
  204 No Content   →  DELETE success
  400 Bad Request  →  Validation error
  401 Unauthorized →  Auth missing/invalid
  403 Forbidden    →  No permission
  404 Not Found    →  Resource missing
  429 Rate Limited →  Too many requests
  500 Server Error →  Backend crash
```

