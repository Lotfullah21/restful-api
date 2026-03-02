# Token Authentication (DRF)

Token authentication is the simplest API auth method in DRF.

Flow:

1. User sends username and password once.
2. Server returns a token.
3. Client sends that token on every protected request.

## Prerequisites

- `djangorestframework` installed
- A Django app with DRF enabled

## Step 1: Enable token auth app

```py
# settings.py
INSTALLED_APPS = [
    # ...
    "rest_framework",
    "rest_framework.authtoken",
]
```

Then run migrations:

```sh
python manage.py migrate
```

## Step 2: Configure DRF authentication class

```py
# settings.py
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework.authentication.TokenAuthentication",
    ],
}
```

## Step 3: Expose token creation endpoint

```py
# urls.py
from django.urls import path
from rest_framework.authtoken.views import obtain_auth_token

urlpatterns = [
    path("auth/token/", obtain_auth_token, name="auth_token"),
]
```

Request:

```http
POST /auth/token/
Content-Type: application/json

{
  "username": "alice",
  "password": "your-password"
}
```

Response:

```json
{
  "token": "1a2b3c..."
}
```

## Step 4: Protect a view

```py
# views.py
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response

@api_view(["GET"])
@permission_classes([IsAuthenticated])
def secret(request):
    return Response({"message": "Only authenticated users can see this."})
```

Use token in header:

```http
Authorization: Token 1a2b3c...
```

## Optional: Role/group check example

```py
@api_view(["GET"])
@permission_classes([IsAuthenticated])
def teachers_view(request):
    if request.user.groups.filter(name="Instructors").exists():
        return Response({"message": "Welcome, instructor."})
    return Response({"message": "You are not in Instructors group."}, status=403)
```

## Common mistakes

- Forgetting `rest_framework.authtoken` in `INSTALLED_APPS`
- Sending `Bearer` instead of `Token` for DRF token auth
- Protecting endpoints with permissions but not setting authentication classes
