## OpenAPI

OpenAPI is a specification
It defines a specific syntax to describe APIs and how to consume them.

- only posts and get can go with plurals
  - `/api/v1/students/`
  - DELETE, PATCH `/api/v1/students/` - WRONG
  - DELETE, PATCH `/api/v1/students/{id}/` - RIGHT

do the state transitions only via dedicate routes, not the general ones
PATCH /resource/{id}/ = descriptive/profile updates only (names, contact, non-lifecycle fields).
PATCH /resource/{id}/status/ = lifecycle state transitions only (status changes with transition rules, required reasons, stricter RBAC/audit).
Create endpoints initialize initial status; later status changes go through /status/ only.

```sh
PATCH /resource/{id}/ = descriptive/profile fields
PATCH /resource/{id}/status/ = lifecycle/state transitions only
```
