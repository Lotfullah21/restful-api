# Best Practices for API Security

## Security Best Practices

### 1. Secure Socket Layer (SSL)

SSL (Secure Sockets Layer), now more commonly referred to as TLS (Transport Layer Security), is a protocol that encrypts data as it travels between the client and the server. Without it, anyone on the same network could intercept and read the data being sent (like passwords or tokens). This is called a "man-in-the-middle" attack.

Using HTTPS (HTTP + SSL/TLS) means all communication is encrypted and cannot be read by a third party. You enable this by installing an SSL certificate on your server. Most cloud providers and services like Let's Encrypt offer free SSL certificates.

### 2. Signed URLs

A signed URL is a URL that includes a cryptographic signature as part of the query string. The server verifies this signature before serving the response. This ensures the URL has not been tampered with and grants temporary, controlled access to a specific resource without requiring the user to be fully logged in.

Example: Use HMAC (Hash-based Message Authentication Code) to generate a signature. HMAC takes a secret key and a message and produces a fixed-size hash. The server recalculates the hash on its side and compares it to the one in the URL. If they match, the request is valid.

### 3. Token-Based Authentication

Instead of sending a username and password with every request, the client authenticates once and receives a token. This token is then sent with every subsequent request to prove identity.

- **JWT (JSON Web Token)**: A compact, self-contained token that encodes user information. It has three parts: a header (token type and algorithm), a payload (user data like ID and role), and a signature (used to verify the token has not been tampered with). The server can verify the token without storing any session data, keeping the system stateless.
- The token is typically sent in the `Authorization` header as: `Authorization: Bearer <token>`.

### 4. HTTP Status Codes for Security

Using the correct HTTP status codes helps clients understand what went wrong during a request:

- `401 Unauthorized`: The request did not include valid credentials. The user is not authenticated — they are not logged in, or they provided a wrong password or an invalid/expired token.
- `403 Forbidden`: The user is authenticated (we know who they are) but they do not have permission to perform the requested action. For example, a regular user trying to access an admin-only endpoint.

### 5. Cross-Origin Resource Sharing (CORS)

CORS is a browser security mechanism that controls which external domains are allowed to make requests to your API. By default, browsers block requests coming from a different domain (origin) than the one that served the web page. This prevents malicious websites from silently making API calls on behalf of a logged-in user.

You configure CORS on the server by setting response headers that specify which origins, HTTP methods, and headers are permitted. For example, you might allow requests only from `https://myfrontend.com` and block all other origins.

### 6. Firewalls

A firewall is a network security system that monitors and filters incoming and outgoing network traffic based on a set of predefined rules. It acts as a barrier between your server and the outside world (the internet), blocking traffic that does not meet your security criteria.

For APIs, you can configure a firewall to:

- Only allow traffic from specific trusted IP addresses (called whitelisting).
- Block known malicious IP addresses.
- Restrict access to certain ports (e.g., only allow traffic on port 443 for HTTPS).

For example, you might configure your firewall so that only your frontend servers and a small number of trusted partners can reach your API, and all other incoming traffic is silently dropped.

## Access Control

### What is Access Control?

Access control defines who can do what in your system. It answers two questions: what actions can a user perform, and what data are they allowed to see or modify?

| Term          | Description                                                                                |
| ------------- | ------------------------------------------------------------------------------------------ |
| **Role**      | A collection of privileges assigned to a type of user (e.g., "admin", "editor", "viewer"). |
| **Privilege** | A specific permission to perform a task (e.g., "can delete posts", "can view reports").    |

### Example:

- **User Role**: Can place an order and view their own profile.
- **Manager Role**: Can view all user data, process refunds, and manage inventory.

In Django, roles and privileges can be configured in the admin panel using Groups and Permissions. In Django REST Framework (DRF), you enforce them in code using permission classes like `IsAuthenticated`, `IsAdminUser`, or custom permission classes.

## Authentication vs Authorization

### Authentication

Authentication is the process of verifying who you are. It answers the question: "Are you really who you claim to be?"

The most common method is checking a username and password. Once verified, the server issues a bearer token (typically a JWT) that the client sends with all future requests to prove its identity without re-entering credentials each time.

### Authorization

Authorization is the process of checking what you are allowed to do. It always happens after authentication and answers the question: "Now that we know who you are, do you have permission to do this specific thing?"

Example:

- A user with the "admin" role can delete resources.
- A user with the "viewer" role can only read data and cannot delete or modify anything.

### Authentication vs Authorization Flow

Here is a step-by-step description of the typical flow:

1. The client sends credentials (username + password) to the login endpoint.
2. The server verifies the credentials (Authentication step).
3. If valid, the server returns a token (e.g., JWT).
4. The client stores the token and sends it in the `Authorization: Bearer <token>` header with every future request.
5. The server verifies the token on each request, then checks if that user has the required permissions for the requested action (Authorization step).
6. If authorized, the server returns the requested data. If not, it returns `403 Forbidden`.
