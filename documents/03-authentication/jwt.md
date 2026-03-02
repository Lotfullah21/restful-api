# JWT Authentication (SimpleJWT)

JWT is a token-based authentication method with two token types:

- Access token: short-lived, sent with API requests
- Refresh token: longer-lived, used to get a new access token

## Step 1: Install package

```sh
pip install djangorestframework-simplejwt
```

## Step 2: Configure Django settings

```py
# settings.py
from datetime import timedelta

INSTALLED_APPS = [
    # ...
    "rest_framework",
    "rest_framework_simplejwt",
    "rest_framework_simplejwt.token_blacklist",  # optional, for logout/blacklist
]

REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework_simplejwt.authentication.JWTAuthentication",
    ],
}

SIMPLE_JWT = {
    "ACCESS_TOKEN_LIFETIME": timedelta(minutes=5),
    "REFRESH_TOKEN_LIFETIME": timedelta(days=1),
    "ROTATE_REFRESH_TOKENS": True,
    "BLACKLIST_AFTER_ROTATION": True,
}
```

Run migrations if you enabled blacklist:

```sh
python manage.py migrate
```

## Step 3: Add JWT endpoints

```py
# urls.py
from django.urls import path
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
    TokenBlacklistView,
)

urlpatterns = [
    path("auth/jwt/create/", TokenObtainPairView.as_view(), name="jwt_create"),
    path("auth/jwt/refresh/", TokenRefreshView.as_view(), name="jwt_refresh"),
    path("auth/jwt/logout/", TokenBlacklistView.as_view(), name="jwt_blacklist"),
]
```

## Step 4: Login and call protected endpoints

Create tokens:

```http
POST /auth/jwt/create/
Content-Type: application/json

{
  "username": "alice",
  "password": "your-password"
}
```

Response:

```json
{
  "refresh": "...",
  "access": "..."
}
```

Use access token in header:

```http
Authorization: Bearer <access-token>
```

Refresh token when access expires:

```http
POST /auth/jwt/refresh/
Content-Type: application/json

{
  "refresh": "<refresh-token>"
}
```

## Common mistakes

- Using `Token` prefix instead of `Bearer` for JWT
- Forgetting `.as_view()` in JWT URL patterns
- Setting very long access token lifetime
- Not enabling blacklist when implementing logout
