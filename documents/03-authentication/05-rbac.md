# Layer 5 — Authorization: RBAC (Role-Based Access Control)

## What Is RBAC?

RBAC stands for **Role-Based Access Control**. The idea is simple:
It is not about identifying users, it is about managing access.
We grant permissions to roles, not to individual users.
user -> role -> permission -> capability

1. Every user has exactly **one role** (e.g., Finance Officer, Branch Manager, Donor)
2. Every role has a **set of permissions** (e.g., "can create disbursement", "can view students")
3. When you try to do something, the system checks: does your role's permission set include this action?

**Why roles instead of giving permissions to each user individually?**

Imagine you have 5 Finance Officers. Without roles, you'd assign 15+ permissions to each one manually. If the rules change, you update all 5 individually. With roles, you define the permissions once on the "Finance Officer" role, and all 5 users inherit them automatically.

### How Django Implements RBAC

Django uses three built-in concepts:

```
User (your account)
  └── belongs to a Group (your role)
        └── Group has Permissions (what you can do)
```

In our system:

| Your Account            | Your Django Group | What You Inherit                                                                               |
| ----------------------- | ----------------- | ---------------------------------------------------------------------------------------------- |
| Ahmad (Finance Officer) | `finance_officer` | Can create fund intakes, disbursements, allocations (status=pending). Cannot approve anything. |
| Fatima (Finance Admin)  | `finance_admin`   | Can create (auto-approved) AND approve others' records                                         |
| Karim (Branch Manager)  | `branch_manager`  | Can register students, view classes — but only for his own branch                              |

**Rule in this system:** One user = one group. No one holds multiple roles. This keeps things simple and avoids conflicts like "what if someone is both a Finance Officer and a Finance Admin?"

## The Four Layers of Permission Checking

When you make an API call, the system checks your permission in **four places**, one after another. All four must pass. Think of it like four locked doors — you need the right key for each one.

### Door 1: DRF View Level — "Can your role do this action?"

This is the first check. Before Django even looks at the data, it asks: does your Django group have the right permission?

Example permissions:

- `finance.add_disbursement` — can create a disbursement
- `finance.approve_disbursement` — can approve a disbursement
- `students.view_student` — can view student records

If your group doesn't have the permission, you get an immediate **403 Forbidden** response. The request stops here.

```
Finance Officer tries to approve a disbursement
 -> Check: does finance_officer group have finance.approve_disbursement?
 -> No!
 -> 403 Forbidden. Request blocked.
```

### Door 2: DRF Serializer Level — "Are the field rules correct for your role?"

Some roles have different rules for the same action. The serializer checks field-level constraints.

Example:

- When a **Finance Officer** creates a record-> `status` is automatically set to `pending` (they can't choose)
- When a **Finance Admin** creates a record-> `status` is automatically set to `approved` (auto-approved by role)

### Door 3: Django ORM Level — "Which records can you see?"

This is **data scoping** (explained in detail below). Even if you have permission to view students, maybe you can only see students from your own branch. The ORM queryset is filtered before data is returned.

### Door 4: Database Level — "Are the hard constraints met?"

The database itself has constraints that can't be bypassed even by code bugs:

- Foreign key constraints (can't reference a branch that doesn't exist)
- CHECK constraints (disbursement amount can't be negative)
- Unique constraints (student can't be enrolled in the same class twice)

### Why Four Layers Instead of One?

Defense in depth. If a bug in your view code accidentally lets an unauthorized request through, the ORM layer still blocks it. If the ORM has a bug, the database constraints still protect the data. No single point of failure.

## Data Scoping — The Other Half of Authorization

Permissions answer: **What actions can you perform?**
Data scoping answers: **Which records can you see?**

Both are required. Here's why:

A Branch Manager and a Training Admin both have `students.view_student` permission. But:

- **Training Admin** sees students from **all 3 branches** (Paghman, Deh Sabz, Baghlan)
- **Branch Manager** sees students from **their own branch only**

Same permission, different data. That's data scoping.

### How Data Scoping Works in This System

| Role            | What They See                                     | How It's Enforced                                |
| --------------- | ------------------------------------------------- | ------------------------------------------------ |
| Branch Manager  | Only records from their own branch                | `queryset.filter(branch_id=user.branch_id)`      |
| Donor           | Only records linked to their own funded portfolio | `queryset.filter(fund_intake__donor_id=user.id)` |
| Finance Officer | All finance records (no scope filter)             | Full queryset                                    |
| Everyone else   | All records in their permitted modules            | Full queryset                                    |

**Important:** Data scoping happens in the Django backend (ORM queryset), not in the frontend. If a Branch Manager somehow bypasses the UI and calls the API directly, they still only get their own branch's data. The frontend hides buttons to make the UI clean, but the real security is on the server.

### Data Scoping Example: Donor

A Donor named "Islamic Relief Foundation" funded a specific fund intake. When they log in and view their dashboard:

```
Without scoping (WRONG — they'd see everything):
  GET /api/finance/disbursements/-> returns ALL disbursements

With scoping (CORRECT):
  GET /api/finance/disbursements/-> returns only disbursements linked
                                     to fund intakes where donor = this user
```

This is why the JWT payload includes the user's `role` and the database tracks which donor is linked to which fund intake. The backend uses this to filter.

## What Each Role Can and Cannot Do

Here's a simplified view. (The full detail is in [role-matrix.md](../../02-architecture/02-security-architecture/role-matrix.md).)

### Platform Admin

- **Can:** Create user accounts, suspend users, reset passwords, reset 2FA, view audit logs
- **Cannot:** Create business data (students, finance records, equipment). They manage the system itself, not the data in it
- **Scope:** Full system (technical access)

### Program Admin

- **Can:** Everything in Students, Finance, Equipment, and Oversight. Create records (auto-approved), approve others' records, configure reports
- **Cannot:** Create user accounts or view audit logs (that's Platform Admin)
- **Scope:** All branches, all modules

### Training Admin

- **Can:** Manage students, faculty, classes, cycles, subjects, equipment categories/items across all branches
- **Cannot:** Access finance module at all. Cannot approve equipment transfers
- **Scope:** All branches (student/equipment modules only)

### Branch Manager

- **Can:** Register students, enroll in classes, update student status, log equipment condition — but **only for their own branch**
- **Cannot:** Create classes, manage faculty, access finance, see other branches
- **Scope:** Own branch only

### Finance Admin

- **Can:** Create financial records (auto-approved), approve Finance Officer's pending records, view all finance data
- **Cannot:** Access student or equipment modules
- **Scope:** All finance data

### Finance Officer

- **Can:** Create financial records — but they're always `status=pending` until someone else approves
- **Cannot:** Approve any records. Cannot approve their own records (this is Segregation of Duties — explained below)
- **Scope:** All finance data (can see everything, but create is limited)

### Independent Monitor

- **Can:** View finance data (read-only). Will verify disbursements in Phase 2
- **Cannot:** Create, update, or delete anything
- **Scope:** Finance module (read-only)

### Executive / Board

- **Can:** View dashboards and aggregated reports across all modules
- **Cannot:** Create, update, or delete anything
- **Scope:** Read-only dashboards

### Donor

- **Can:** View their own funded portfolio — fund intakes, allocations, disbursements, repayments linked to their donations
- **Cannot:** See other donors' data, see aggregated totals, modify anything
- **Scope:** Own funded portfolio only

## Segregation of Duties (SoD) — Why Roles Are Separated

SoD is a principle that says: **the person who creates a record should not be the same person who approves it**.

Why? To prevent fraud. If one person can both create and approve financial records, they could create fake disbursements and approve them without anyone else checking.

### How SoD Works in This System

```
Finance Officer creates a disbursement
 -> status = "pending"
 -> Finance Officer CANNOT approve it (code checks: creator ≠ approver)

Finance Admin reviews it
 -> Finance Admin APPROVES it
 -> status = "approved"
```

The code enforces this:

```python
def approve(self, request, pk=None):
    record = self.get_object()
    if record.created_by == request.user:
        raise PermissionDenied("Cannot approve your own record.")
    record.status = "approved"
    record.approved_by = request.user
    record.save()
```

**Exception:** Finance Admin and Program Admin are "trusted roles." When they create a record, it's auto-approved (`status=approved` on creation). This is a deliberate design decision — these are senior roles with oversight responsibility.

## Why Django Checks Permissions on Every Request (Not Just at Login)

You might think: "I logged in and got my role. Why does Django check my permissions again on every API call?"

Three reasons:

1. **Roles can change.** A Platform Admin might change your role from Finance Admin to Finance Officer while you're logged in. Your JWT token still says your old role, but the backend re-checks the database on every request.

2. **Accounts can be suspended.** If a Platform Admin suspends your account (`is_active = False`), your existing token is still valid for up to 30 minutes. But the backend checks `is_active` on every request and rejects it immediately.

3. **JWT claims are convenience, not security.** The `role` field in the JWT payload is there so the frontend knows which screens to show. But the backend NEVER trusts the JWT role for permission decisions — it always looks up the user's actual group in the database. This means even if someone tampered with the JWT payload, the backend would still enforce the correct permissions.

**Key principle:** The JWT gets you through the door (authentication). But once inside, the database decides what rooms you can enter (authorization). Every. Single. Time.

## Putting It All Together — Complete Flow

Here's what happens when a Finance Officer tries to create a disbursement:

```
1. Finance Officer sends:
   POST /api/finance/disbursements/
   Headers: Authorization: Bearer <access_token>
   Body: { beneficiary_id: 5, amount: 2500, evidence_file: ... }

2. AUTHENTICATION (Layers 1-4):
  -> Is the JWT valid? Yes
  -> Is it expired? No (within 30 minutes)
  -> Is the user active? Yes (database check)
  -> Authentication passed

3. AUTHORIZATION — Door 1 (View Permission):
  -> User's group: finance_officer
  -> Required permission: finance.add_disbursement
  -> Does finance_officer group have it? Yes
  -> Permission check passed

4. AUTHORIZATION — Door 2 (Serializer Rules):
  -> User is Finance Officer, not Finance Admin
  -> Status is automatically set to "pending" (cannot choose "approved")
  -> Field rules applied

5. AUTHORIZATION — Door 3 (Data Scoping):
  -> Finance Officer has no data scope restriction
  -> No filtering needed for create

6. AUTHORIZATION — Door 4 (Database Constraints):
  -> Does beneficiary_id=5 exist? Yes
  -> Does beneficiary have an approved allocation? Yes
  -> Is amount ≤ remaining allocation? Yes
  -> Database constraints passed

7. RESULT:
  -> Disbursement created with status = "pending"
  -> Audit log entry written
  -> Response: 201 Created
```

Now if the same Finance Officer tries to approve it:

```
1. POST /api/finance/disbursements/42/approve/

2. AUTHENTICATION: (same as above)

3. AUTHORIZATION — Door 1:
  -> Required permission: finance.approve_disbursement
  -> Does finance_officer group have it? NO
  -> 403 Forbidden. Request blocked immediately.
```

## The Answers to Layer 5 Questions

Now you should be able to answer these:

**Q: What is the difference between authentication and authorization?**
Authentication = proving who you are (password + 2FA-> JWT token). Authorization = the system deciding what you're allowed to do (role-> permissions-> data scoping).

**Q: In this system, what does a Branch Manager see that a Finance Officer cannot?**
Branch Manager can see students, classes, enrollments, faculty, and equipment — but only for their own branch. Finance Officer cannot see any of these — they only have access to the finance module. The Branch Manager cannot see finance data; the Finance Officer cannot see student data. They operate in completely separate modules.

**Q: Why does Django check permissions on every request, not just at login?**
Because roles can change mid-session, accounts can be suspended, and JWT claims are convenience (not a trust boundary). The backend always re-validates from the database. This prevents stale permissions from being used.

**Q: What is "data scoping" and why does a Donor need it?**
Data scoping filters which records a user can see, beyond just which actions they can perform. A Donor needs it because they should only see their own funded portfolio — not other donors' fund intakes, disbursements, or repayments. Without data scoping, every Donor would see every other Donor's data, which is a privacy and trust violation.

## Key Takeaways

1. **Authentication is the door. Authorization is the room.** Getting through the door doesn't mean you can open every drawer inside.

2. **RBAC makes permissions manageable.** Define permissions once per role, inherit automatically per user.

3. **Permissions + Data Scoping = Full authorization.** Permissions control actions. Data scoping controls visibility. Both are enforced server-side.

4. **Defense in depth.** Four layers of checking (view-> serializer-> ORM-> database) means no single bug can break security.

5. **SoD prevents fraud.** The person who creates a record cannot approve it. This is enforced in code, not by trust.

6. **Never trust the frontend.** The frontend hides buttons for convenience. The backend enforces rules for security.

## Where This Fits in the Full Picture

```
Layer 1 — HTTP is stateless. We need to identify users.
  └── Layer 2 — JWT is a signed token that identifies you.
       └── Layer 3 — Login gives you tokens. Refresh keeps you alive. Logout kills them.
            └── Layer 4 — 2FA adds a second factor for sensitive roles.
                 └── Layer 5 — RBAC determines what you can do once you're in. ← YOU ARE HERE
```
