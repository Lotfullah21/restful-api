# Permissions and Access Control

Authentication checks identity. Permissions check access level.

This file shows how to allow everyone to read, but restrict write actions to staff/admin users.

## Step 1: Create a custom permission

```py
# permissions.py
from rest_framework import permissions

class IsStaffOrSuperuserForWrite(permissions.BasePermission):
    """Allow read-only access to everyone, write access to staff/superusers."""

    message = "Only staff users can modify this resource."

    def has_permission(self, request, view):
        if request.method in permissions.SAFE_METHODS:
            return True
        return bool(
            request.user
            and request.user.is_authenticated
            and (request.user.is_staff or request.user.is_superuser)
        )
```

## Step 2: Apply it to a view

```py
# views.py
from rest_framework import generics
from .models import Course
from .serializers import CourseSerializer
from .permissions import IsStaffOrSuperuserForWrite

class CourseList(generics.ListCreateAPIView):
    queryset = Course.objects.all()
    serializer_class = CourseSerializer
    permission_classes = [IsStaffOrSuperuserForWrite]
```

Behavior:

- `GET` is open to all users
- `POST`, `PUT`, `PATCH`, `DELETE` require authenticated staff/superuser

## Optional: combine with built-in permissions

If you want read-only for anonymous users and write access for any authenticated user:

```py
from rest_framework.permissions import IsAuthenticatedOrReadOnly

permission_classes = [IsAuthenticatedOrReadOnly]
```

## Quick test checklist

1. Send `GET` without token -> should work.
2. Send `POST` without token -> should return `401`.
3. Send `POST` with non-staff token -> should return `403`.
4. Send `POST` with staff/superuser token -> should succeed.

## Common mistakes

- Checking only `is_staff` and forgetting `is_authenticated`
- Using string `"GET"` instead of `permissions.SAFE_METHODS`
- Expecting auth classes alone to enforce role-based rules
