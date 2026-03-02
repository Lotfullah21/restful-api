# Djoser Authentication

Djoser provides ready-made authentication endpoints for DRF.

It can handle:

- User registration
- User activation
- Login/logout (token or JWT, depending on config)
- Password reset flows

## Step 1: Install dependencies

```sh
pip install djoser
```

Install one auth backend option:

```sh
# Option A: DRF token auth
pip install djangorestframework

# Option B: JWT auth
pip install djangorestframework-simplejwt
```

## Step 2: Configure settings

```py
# settings.py
INSTALLED_APPS = [
    # ...
    "rest_framework",
    "djoser",
]

REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework.authentication.TokenAuthentication",
        # or: "rest_framework_simplejwt.authentication.JWTAuthentication",
    ],
}

DJOSER = {
    "LOGIN_FIELD": "email",  # or "username"
    "USER_CREATE_PASSWORD_RETYPE": True,
    "SEND_ACTIVATION_EMAIL": False,
}
```

If you use token auth endpoints, add this too:

```py
INSTALLED_APPS += ["rest_framework.authtoken"]
```

Then migrate:

```sh
python manage.py migrate
```

## Step 3: Add URLs

```py
# urls.py
from django.urls import include, path

urlpatterns = [
    path("auth/", include("djoser.urls")),
    path("auth/", include("djoser.urls.authtoken")),  # token endpoints
    # path("auth/", include("djoser.urls.jwt")),      # JWT endpoints (use instead of authtoken)
]
```

## Commonly used endpoints

- `POST /auth/users/` create user
- `GET /auth/users/me/` current user
- `POST /auth/token/login/` login (authtoken mode)
- `POST /auth/token/logout/` logout (authtoken mode)

## Example: register user

```http
POST /auth/users/
Content-Type: application/json

{
  "email": "alice@example.com",
  "username": "alice",
  "password": "StrongPass123!",
  "re_password": "StrongPass123!"
}
```

## Common mistakes

- Installing `filter-backends` (not required for Djoser)
- Enabling both JWT and authtoken endpoints without a clear reason
- Forgetting to align `LOGIN_FIELD` with your user model
