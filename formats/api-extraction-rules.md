# How to Extract API Endpoints

## The Rule

**Use a hybrid flow: start from user actions, map to entities, enforce rules.**

```
User Stories / Features  →  what API consumers need to do
Entities (ERD)           →  what resources exist
Business Rules + States  →  what is allowed and how state can change
```

## Step-by-Step

**Step 1 — List user actions from stories/features**

Write the actions users must perform (register, approve, transfer, repay, etc.).

**Step 2 — List entities from the ERD**

Each entity becomes a resource candidate in the API.

**Step 3 — Give each resource base CRUD**

```
GET    /api/v1/{entity}/           list all
GET    /api/v1/{entity}/{id}/      get one
POST   /api/v1/{entity}/           create
PATCH  /api/v1/{entity}/{id}/      partial update
DELETE /api/v1/{entity}/{id}/      soft delete
```

**Step 4 — Apply business rules to remove what's not allowed**

| Rule                               | What to remove |
| ---------------------------------- | -------------- |
| Entity is immutable after creation | Remove PATCH   |
| Entity is non-deletable            | Remove DELETE  |
| Entity has no update concept       | Remove PATCH   |

Examples for this project:

- Repayment → immutable → remove PATCH and DELETE
- Evidence File → non-deletable → remove DELETE
- Donor Fund Intake → immutable → remove PATCH and DELETE

**Step 5 — Check for status/lifecycle → add `/status/` endpoint**

If the entity has a state machine (e.g. `pending → approved`, `active → suspended`), add:

```
PATCH  /api/v1/{entity}/{id}/status/
```

Why separate from PATCH? Because status transitions have different rules — they may require a reason, trigger audit logs, or enforce allowed transitions. Mixing them with regular field updates creates confusion.

**Step 6 — Check for parent-child relationships → nest where it makes sense**

If entity B always belongs to entity A and is never accessed independently:

```
GET   /api/v1/{entityA}/{id}/{entityB}/    list B under A
POST  /api/v1/{entityA}/{id}/{entityB}/    create B under A
```

Example: disbursements always belong to a beneficiary:

```
GET  /api/v1/beneficiaries/{id}/disbursements/
POST /api/v1/beneficiaries/{id}/disbursements/
```

**Step 7 — Add non-CRUD action endpoints from user actions**

If a user story describes a verb that is not CRUD (transfer, approve, verify, reverse):

```
POST  /api/v1/{entity}/{id}/{action}/
```

Examples:

```
POST  /api/v1/allocations/{id}/approve/
POST  /api/v1/students/{id}/transfer/
```

## Summary Table

| Source                                  | Produces                  |
| --------------------------------------- | ------------------------- |
| User stories / feature actions          | Required API capabilities |
| Entity in ERD                           | Base resource endpoints   |
| Business rule (immutable/non-deletable) | Removes PATCH or DELETE   |
| Status field with lifecycle             | `/status/` sub-endpoint   |
| Parent-child relationship               | Nested resource endpoint  |
| Business action (verb in user story)    | Custom action endpoint    |

## What NOT to do

- Do not create endpoints for every attribute — attributes are fields in the request/response body, not endpoints
- Do not create endpoints for computed fields (e.g. `outstanding_balance`) — return them in the GET response
- Do not create separate endpoints for filtering — use query parameters on the list endpoint
- Do not create endpoints that don't trace back to a user story or business rule
