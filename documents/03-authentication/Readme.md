# Authentication

## Layer 1 — The Problem: Why Authentication Exists

**What you need to understand:**

- HTTP is stateless — every request is anonymous by default
- Without auth, anyone can call any API endpoint
- The question auth answers: "Who are you, and are you allowed to do this?"

**How to learn it:**

1. Watch: [HTTP Cookies & Sessions — Fireship (7 min)](https://www.youtube.com/watch?v=UBUNrFtufWo)
2. Read: [MDN — HTTP authentication overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication) (first two sections only)

**Answer these before moving to Layer 2:**

- Why does a server need to "remember" you between requests?
- What is a session? What is a cookie?
- Why did the industry move away from sessions toward tokens?

## Layer 2 — Tokens: What Is a JWT?

**What you need to understand:**

- A JWT is a self-contained, signed string that proves who you are
- It has 3 parts: header, payload, signature
- The server issues it. You send it back on every request. The server verifies the signature.

**How to learn it:**

1. Watch: [JWT Explained — Fireship (100 seconds)](https://www.youtube.com/watch?v=SqcY0GlETPk)
2. Go to [jwt.io](https://jwt.io) — paste the example token in the debugger, see the three parts decoded live
3. Read: [JWT Introduction — jwt.io/introduction](https://jwt.io/introduction) (3 min)

**Answer these before moving to Layer 3:**

- What are the 3 parts of a JWT?
- Why can't someone fake the payload?
- Why does the token expire?
- What is the difference between `access token` and `refresh token`?

## Layer 3 — The Full Login Flow

**What you need to understand:**

- Login → server verifies password → issues access token + refresh token
- Client stores tokens, sends access token on every API call
- When access token expires, client uses refresh token to get a new one silently
- Logout = blacklist the refresh token

**How to learn it:**

1. Watch: [Access Tokens vs Refresh Tokens — ByteByteGo (6 min)](https://www.youtube.com/watch?v=vSzjgWt8OqM)
2. Read your own document: [01-auth-flow.md](../../03-api/02-api/05-auth/01-auth-flow.md) — Flows 1 through 4

**Answer these before moving to Layer 4:**

- Why is the access token short-lived (30 min) but the refresh token long-lived (24 hr)?
- What happens when someone steals your access token?
- What happens when someone steals your refresh token?
- Why do refresh tokens rotate?

## Layer 4 — Two-Factor Authentication (2FA / TOTP)

**What you need to understand:**

- Password alone can be stolen (phishing, data breach, keylogger)
- 2FA adds a second factor: something you _have_ (your phone) + something you _know_ (password)
- TOTP = Time-based One-Time Password — a 6-digit code that changes every 30 seconds
- Google Authenticator generates TOTP codes locally — no internet needed

**How to learn it:**

1. Watch: [Two Factor Auth Explained — Fireship (100 seconds)](https://www.youtube.com/watch?v=iXSyxm9jmmo)
2. Read: [How TOTP works — Wikipedia](https://en.wikipedia.org/wiki/Time-based_one-time_password#Algorithm) (Algorithm section only)
3. Read your own document: [01-auth-flow.md](../../03-api/02-api/05-auth/01-auth-flow.md) — Flows 2, 7, 8

**Answer these before moving to Layer 5:**

- Why is TOTP more secure than SMS codes?
- Why does the code expire every 30 seconds?
- What is a `temp_token` and why does it exist between step 1 and step 2 of login?
- Why do only Finance and Admin roles need 2FA in this system?

## Layer 5 — Authorization: RBAC (Role-Based Access Control)

**What you need to understand:**

- Authentication = who you are
- Authorization = what you are allowed to do
- RBAC = your role determines your permissions
- In Django: Users → Groups → Permissions

**How to learn it:**

1. Watch: [RBAC Explained — ByteByteGo (5 min)](https://www.youtube.com/watch?v=4Uya_I_Oxjk)
2. Read your own document: [auth-design.md §6](../../02-architecture/02-security-architecture/auth-design.md) — Role-to-Group Mapping and Permission Enforcement sections

**Answer these after Layer 5:**

- What is the difference between authentication and authorization?
- In this system, what does a Branch Manager see that a Finance Officer cannot?
- Why does Django check permissions on every request, not just at login?
- What is "data scoping" and why does a Donor need it?

## The Order In One View

```
Layer 1 — Why auth exists (HTTP stateless, session problem)
  └─ Layer 2 — What is a JWT (tokens, signature, expiry)
       └─ Layer 3 — Full login flow (access + refresh, rotation, blacklist)
            └─ Layer 4 — 2FA / TOTP (second factor, temp_token, TOTP codes)
                 └─ Layer 5 — RBAC (role → permissions → data scoping)
```
