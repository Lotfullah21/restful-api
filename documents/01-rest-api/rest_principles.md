# Best Practices for API Development

## General Best Practices

### 1. Keep it Simple

The API design should be intuitive and straightforward. Use clear, predictable naming for endpoints, avoid unnecessary complexity, and make it easy for other developers (or your future self) to understand what each endpoint does just by reading its URL and method.

### 2. One API, One Job

Each API endpoint should handle a single, clear purpose. For example, an endpoint that lists users should only list users, not also handle creating them. This principle is similar to the Single Responsibility Principle in software design — each piece of code should do one thing well.

### 3. Include Filtering, Ordering, and Pagination

When an endpoint returns a list of items, it should support ways to narrow down, sort, and page through results:

- **Filtering**: Return only items that match a condition.
  - Example: `/users?role=admin` returns only admin users.
- **Ordering**: Sort results by a field.
  - Example: `/users?sort=created_at` sorts users by when they were created.
- **Pagination**: Return results in chunks (pages) instead of all at once.
  - Example: `/users?page=1&limit=10` returns the first 10 users.

Without pagination, an endpoint returning thousands of records would be very slow and wasteful of server resources.
These values (for example `role=admin`, `sort=created_at`, `page=1`) are query parameters inside the query string. See [query_string_and_params.md](../02-http-https/query_string_and_params.md).

### 4. Make the API Cacheable

Caching means storing a copy of a response temporarily so it can be returned quickly on future identical requests without hitting the database again. HTTP cache headers (like `Cache-Control` and `ETag`) tell the client or intermediate servers how long they can reuse a stored response. This reduces server load and speeds up the experience for the user.

### 5. Implement Rate Limiting

Rate limiting restricts how many requests a single client (user or IP address) can make within a given time window. For example, allowing a maximum of 100 requests per minute per user. This protects the API from being overwhelmed by abuse, bots, or accidental infinite loops in client code.

### 6. Monitor Latency

Latency is the time it takes for a request to travel from the client to the server and receive a response back. High latency means a slow API, which leads to a poor user experience. Regularly tracking latency helps you detect performance problems early, such as slow database queries or bottlenecks in your code.
