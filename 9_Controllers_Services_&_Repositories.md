# 🏗️ Backend System Design: Controllers, Services, & Repositories

A comprehensive technical guide to the internal architecture of a modern backend server, detailing the request life cycle, middleware chains, and state management.

---

## 1. The Internal Request Life Cycle
When an HTTP request hits a server, it doesn't just jump to the database. it follows a rigorous, multi-step pipeline. Understanding this flow is essential for debugging and performance optimization.



### The Journey of a Request:
1.  **OS Level:** The Operating System receives packets on a port (e.g., 80, 443, 3000).
2.  **Entry Point:** The web framework (Express, Go Gin, FastAPI) picks up the raw request.
3.  **Routing:** The router matches the URL and HTTP Method to a specific **Handler**.
4.  **Middleware Chain:** The request passes through security and logging filters.
5.  **Handler/Controller:** Data is extracted and validated.
6.  **Service Layer:** Business logic is applied.
7.  **Repository Layer:** The database is queried.
8.  **Response:** The data travels back up the chain to the client.

---

## 2. Layered Architecture (Separation of Concerns)

To maintain a clean codebase, we divide responsibilities into three distinct layers. This is known as **Three-Tier Logic**.

### A. The Controller Layer (The "Gatekeeper")
The Controller is the only part of the application that "knows" about HTTP. 
* **Responsibilities:**
    * **Binding:** Taking JSON/Query strings and converting them to native objects.
    * **Validation:** Checking if required fields exist (e.g., Is the email valid?).
    * **Response Formatting:** Sending the correct HTTP Status Code (201 Created, 400 Bad Request, 404 Not Found).
* **Golden Rule:** A controller should never contain business logic (like calculating a discount or processing a payment). It should only delegate to a Service.

### B. The Service Layer (The "Brain")
This layer contains the core business logic of your application.
* **Responsibilities:**
    * **Orchestration:** Calling multiple repositories (e.g., "Find User" then "Check Balance").
    * **External Integration:** Calling 3rd party APIs (Stripe, Twilio, AWS).
    * **Agnostic Nature:** A service doesn't know if it's being called by a Web API, a Mobile App, or a Cron Job.
* **Golden Rule:** Keep this layer "pure." Avoid using HTTP objects (like `req` or `res`) inside services.

### C. The Repository Layer (The "Data Librarian")
This layer encapsulates all database-specific code.
* **Responsibilities:**
    * **Query Construction:** Writing SQL, NoSQL, or ORM queries.
    * **Data Mapping:** Returning raw data from the DB back to the Service.
* **Golden Rule:** One repository method should do one thing (Single Responsibility). Don't mix "Update User" and "Send Email" in the same repository.

---

## 3. Middleware: The Request Pipeline
Middlewares are optional functions that execute in the "middle" of the request-response cycle. They act as filters.



### Standard Middleware Stack (Order Matters):
1.  **CORS & Security:** Handled first to block unauthorized domains.
2.  **Logging:** Captures the start time of the request.
3.  **Authentication:** Checks the JWT/Session. If invalid, it terminates the request immediately (401 Unauthorized).
4.  **Rate Limiting:** Prevents DDoS by checking the Request Counter per IP.
5.  **Data Parsing:** Converts raw buffers into JSON objects.
6.  **Global Error Handler:** Positioned at the very end to catch any "crashes" and return a clean JSON error instead of a stack trace.

---

## 4. Request Context: The Shared State
The **Request Context** is a scoped "backpack" for a single request. It allows information to flow between Middlewares and Handlers without "tight coupling."

### Real-World Use Case:
1.  **Auth Middleware** extracts a `user_id` from a token.
2.  It saves that `user_id` into the **Context**.
3.  **Service Layer** later reads that `user_id` from the Context to ensure the user is only deleting their *own* posts.

> **Key Benefit:** You don't have to pass the `user_id` as a parameter through 10 different functions; it's always available in the shared Context object.

---

## 5. Summary Table for Quick Reference

| Layer | Input | Output | Primary Goal |
| :--- | :--- | :--- | :--- |
| **Controller** | HTTP Request | HTTP Response | Data I/O & Routing |
| **Service** | Native Objects | Logic Results | Business Intelligence |
| **Repository** | Raw Query Data | Data Entity | Data Persistence |
| **Middleware** | Request/Next() | Modified Req / Error | Pre-processing & Security |

