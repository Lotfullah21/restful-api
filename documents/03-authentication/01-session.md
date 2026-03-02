# Authentication Fundamentals — Layer 1 Study Notes

A structured guide to understanding why authentication exists, how sessions and cookies work, and why JWTs replaced session-based auth at scale.

## Why Does a Server Need to Remember You?

HTTP is **stateless and anonymous**. Every request arrives at the server as a blank stranger — no history, no identity, no context.

Without authentication, a request like:

```
GET /api/v1/finance/disbursements/
```

gives the server no way to know:

- Who is making this request?
- Are they a Finance Officer, a Donor, or a random person on the internet?
- What data are they allowed to see?
- What should the audit log say?

**Authentication solves this:** it attaches a proven identity to every request so the server can filter data, enforce permissions, and write accurate audit logs.

## Sessions and Cookies

### What is a Session?

A **session** is a record stored on the server (in memory or a database):

```
Session Record:
  session_id: "abc123xyz"
  user_id:    42
  role:       "finance_officer"
  logged_in:  2024-02-19T14:30:00Z
```

### What is a Cookie?

A **cookie** is a small piece of data stored in the browser that is sent automatically with every request to the same domain. It typically contains just the session ID — a random string like `abc123xyz`.

### The Flow

```
1. User logs in
   → Server creates a session record
   → Server sends back a cookie containing the session_id

2. Every future request
   → Browser automatically sends the cookie

3. Server receives the cookie
   → Looks up session_id in the database
   → Finds who you are and what you're allowed to do
```

**Key distinction:** The cookie is the _ID card_. The session is the _actual data_.

## Why Move from Sessions to Tokens?

### The Problem at Scale

Sessions work fine with a single server. But modern applications run on many servers behind a load balancer:

```
User logs in → hits Server A → session stored on Server A

Next request → load balancer routes to Server B
Server B looks up the session → NOT FOUND → user is logged out
```

To fix this with sessions, every server needs access to a **shared session store** (like Redis). That's extra infrastructure, extra complexity, extra cost.

### The Token Solution

A **token** (specifically a JWT — JSON Web Token) is **self-contained**. The server puts your identity _directly inside_ the token:

```json
{
	"user_id": 42,
	"role": "finance_officer",
	"exp": 1708300000
}
```

Then it **signs** the token with a secret key. Any server that knows the secret key can verify the token is genuine — without any database lookup.

```
User logs in → Server A issues a signed token → gives it to the browser

Next request → browser sends token → Server B receives it
Server B verifies the signature (no DB lookup) → knows who you are
```

### Why the Signature Matters

This is what **"JWT is stateless"** means in practice:

- The server doesn't need to _store_ anything to verify you
- It just checks: _was this token signed with our secret key?_
- If someone tampers with the payload (e.g. changes `"role": "donor"` to `"role": "platform_admin"`), the signature **breaks** and the server rejects the token

This is why you must **never expose your secret key**.

## Summary

| Concept     | What it is                             | Where it lives                   |
| ----------- | -------------------------------------- | -------------------------------- |
| **Session** | Server-side record of a logged-in user | Server memory / database         |
| **Cookie**  | Carrier for the session ID             | Browser (sent automatically)     |
| **JWT**     | Self-contained, signed identity token  | Browser (sent in request header) |

**Why tokens beat sessions at scale:** tokens are verified by signature, not by database lookup — so any server can independently verify any token with no shared state required.
