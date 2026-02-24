# HTTP Methods (HTTP Verbs)

HTTP methods tell the server what action to perform on a resource. In REST APIs, these methods map directly to the four basic operations you can perform on data, often called CRUD: Create, Read, Update, Delete.

Think of a resource like a record in a database (e.g., a course). The URL tells the server which resource you are talking about, and the HTTP method tells it what to do with it.

## The Five Core HTTP Methods

### 1. GET — Read a Resource

GET is used to retrieve data from the server. It does not modify anything — it is a "read-only" operation. You can safely call GET as many times as you want and the data will not change.

- Fetching a list: `GET /courses` — returns all courses
- Fetching one item: `GET /courses/1` — returns the course with ID 1

No request body is needed. You can pass optional filters as query parameters:

```
GET /courses?category=programming&page=1
```

**Response:** `200 OK` with the requested data in the body.

### 2. POST — Create a Resource

POST is used to create a new resource on the server. You send the new data in the request body (usually as JSON). Each POST call creates a new record.

- Creating a course: `POST /courses`

Request body example:

```json
{
	"title": "Python for Beginners",
	"duration": 30,
	"category": "programming"
}
```

**Response:** `201 Created` with the newly created resource in the body.

### 3. PUT — Replace a Resource Entirely

PUT is used to completely replace an existing resource. You must send the full data for the resource, not just the fields you want to change. Whatever you send becomes the new version of the resource.

- Replacing a course: `PUT /courses/1`

Request body example (all fields required):

```json
{
	"title": "Advanced Python",
	"duration": 45,
	"category": "programming"
}
```

If you only send some fields, the missing ones may be erased or set to null. Use PATCH instead if you only want to update part of the resource.

**Response:** `200 OK` with the updated resource.

### 4. PATCH — Partially Update a Resource

PATCH is used to update only specific fields of an existing resource. Unlike PUT, you only send the fields you want to change — everything else stays the same.

- Partially updating a course: `PATCH /courses/1`

Request body example (only the fields to change):

```json
{
	"duration": 35
}
```

This changes only the `duration` field. The `title` and `category` remain unchanged.

**Response:** `200 OK` with the updated resource.

### 5. DELETE — Remove a Resource

DELETE is used to remove a resource from the server. Once deleted, the resource is gone (unless your server has soft-delete logic).

- Deleting a specific course: `DELETE /courses/1`
- Deleting all courses: `DELETE /courses` (use with caution — some APIs restrict this)

No request body is typically needed.

**Response:** `204 No Content` (success, nothing to return) or `200 OK` with a confirmation message.

## PUT vs PATCH — What is the Difference?

This is a common point of confusion:

| Aspect          | PUT                          | PATCH                         |
| --------------- | ---------------------------- | ----------------------------- |
| What it does    | Replaces the entire resource | Updates only specified fields |
| Fields required | All fields must be sent      | Only the fields to change     |
| Missing fields  | May be erased or set to null | Unchanged                     |
| Use case        | Full replacement             | Partial update                |

Example: If a course has `title`, `duration`, and `category`:

- A PUT with only `{ "title": "New Title" }` would erase `duration` and `category`.
- A PATCH with only `{ "title": "New Title" }` would only update `title`, leaving the rest.

## HTTP Method to CRUD Mapping

| HTTP Method | CRUD Operation   | Example                            |
| ----------- | ---------------- | ---------------------------------- |
| GET         | Read             | `GET /courses` or `GET /courses/1` |
| POST        | Create           | `POST /courses`                    |
| PUT         | Update (replace) | `PUT /courses/1`                   |
| PATCH       | Update (partial) | `PATCH /courses/1`                 |
| DELETE      | Delete           | `DELETE /courses/1`                |

## Idempotency

Idempotency means calling the same request multiple times produces the same result as calling it once. This is an important concept in REST.

- **GET** is idempotent: calling `GET /courses/1` ten times always returns the same course.
- **PUT** is idempotent: replacing a course with the same data ten times results in the same state.
- **DELETE** is idempotent: deleting a course that is already deleted should return `404 Not Found`, but the end state (resource gone) is the same.
- **POST** is NOT idempotent: calling `POST /courses` ten times with the same body creates ten separate courses.
- **PATCH** is generally not idempotent, though it can be depending on implementation.

Understanding idempotency helps you design APIs that are safe to retry (e.g., if a network request times out and you are not sure if it was received).

### Why Idempotency is Critical in Financial Flows

Consider this scenario: a user clicks "Pay Now". The request is sent to the server, but the network drops before the response arrives. The client does not know if the payment went through. If it retries, and the server has no protection, the customer gets charged twice.

This is a real problem. To solve it, APIs use an **Idempotency Key**.

#### What is an Idempotency Key?

An idempotency key is a unique identifier (usually a UUID) that the client generates and sends with the request — typically in a custom HTTP header like `Idempotency-Key`. The server stores this key along with the result of the operation.

- If the server receives a request with a key it has never seen before, it processes it normally and saves the result against that key.
- If the server receives the same key again (a retry), it does NOT process the request again. It simply returns the previously saved result.

This means the operation runs exactly once, no matter how many times the client retries.

#### How it Works Step by Step

```
1. Client generates a unique key:   key = "a1b2-c3d4-e5f6"
2. Client sends the payment:        POST /payments
                                    Idempotency-Key: a1b2-c3d4-e5f6
                                    { "amount": 100, "currency": "USD" }

3. Network drops. Client does not know if it succeeded.

4. Client retries with the SAME key: POST /payments
                                     Idempotency-Key: a1b2-c3d4-e5f6
                                     { "amount": 100, "currency": "USD" }

5. Server looks up "a1b2-c3d4-e5f6" in its store.
   - Found: return the original response. No charge is made again.
   - Not found: process the payment, store the result under this key.
```

#### What the Server Stores

The server needs a small database table (or cache) to track processed idempotency keys:

| Key            | Result                                         | Created At       | Expires At       |
| -------------- | ---------------------------------------------- | ---------------- | ---------------- |
| a1b2-c3d4-e5f6 | `{"status": "success", "charge_id": "ch_001"}` | 2024-01-01 10:00 | 2024-01-02 10:00 |

Keys are typically stored for 24 hours. After expiry, the same key from the same client would be treated as a new request.

#### Rules for Using Idempotency Keys

- The client always generates the key — never the server. This is because the client needs the key before sending the request.
- The key must be unique per operation. Do not reuse the same key for two different payments.
- If the request body changes but the key is the same, the server should reject it with `422 Unprocessable Entity` or `409 Conflict`, because the client is trying to do something different with an already-used key.
- Only apply idempotency keys to non-idempotent methods (POST, PATCH). GET, PUT, and DELETE are already idempotent by nature.

#### Real-World Example: Stripe

Stripe (a payment API) implements this exactly. Every POST request to create a charge accepts an `Idempotency-Key` header. If you send the same key twice, Stripe returns the original charge result without creating a second one. This is considered the industry standard for financial APIs.

## Safe Methods

A method is called "safe" if it does not change any data on the server. Safe methods are read-only.

- **Safe:** GET
- **Not safe:** POST, PUT, PATCH, DELETE

## Common HTTP Status Codes Per Method

| Method | Success Code     | Notes                        |
| ------ | ---------------- | ---------------------------- |
| GET    | `200 OK`         | Returns the resource(s)      |
| POST   | `201 Created`    | Returns the new resource     |
| PUT    | `200 OK`         | Returns the updated resource |
| PATCH  | `200 OK`         | Returns the updated resource |
| DELETE | `204 No Content` | Nothing to return            |

If the resource is not found, all methods return `404 Not Found`. If the user is not authorized, they get `401 Unauthorized` or `403 Forbidden`.
