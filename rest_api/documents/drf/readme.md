# Django REST Framework (DRF)

## What is DRF?

Django REST Framework (DRF) is a third-party library built on top of Django that makes it easy to build RESTful APIs. Django on its own is a web framework for building full web applications (with HTML pages, forms, templates, etc.). DRF extends Django specifically for building APIs that return data (usually JSON) instead of HTML pages.

Think of it this way:

- Django = the foundation (handles routing, database, models, etc.)
- DRF = the toolkit on top of Django for building APIs

Without DRF you would have to manually parse request data, convert database objects to JSON, handle authentication, and write a lot of boilerplate code. DRF handles all of this for you.

## What Problem Does DRF Solve?

When a client (like a React frontend or a mobile app) talks to your server, it sends and receives data — not HTML pages. The typical flow is:

1. Client sends a request (e.g., `GET /api/courses`)
2. Django receives the request and queries the database
3. The database returns Python objects (Django model instances)
4. These Python objects must be converted to JSON so the client can read them
5. The JSON is sent back as the response

Step 4 — converting Python objects to JSON — is called **serialization**. DRF provides serializers, views, and authentication tools to handle this entire process cleanly and consistently.

## Installation

```sh
pip install djangorestframework
```

Then add it to your `INSTALLED_APPS` in `settings.py`:

```py
# settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    # ... other apps
    'rest_framework',   # add this
]
```

## Core Concepts

### Serialization

Serialization is the process of converting a Python object (like a Django model instance) into a format that can be transmitted over a network — usually JSON. The reverse — converting JSON back into a Python object — is called deserialization.

Example: You have a `Course` model in your database. A serializer converts a `Course` instance like this:

```py
# Python object (in memory)
<Course: id=1, title="Python Basics", duration=30>

# After serialization (sent over the network as JSON)
{"id": 1, "title": "Python Basics", "duration": 30}
```

### DRF's Request and Response Objects

DRF wraps Django's built-in `HttpRequest` and `HttpResponse` with its own `Request` and `Response` objects that have extra capabilities:

- `request.data` — works for JSON, form data, and multipart (file uploads). Django's `request.POST` only works for form data.
- `Response(data)` — automatically renders the response as JSON (or XML, etc.) depending on what the client asked for.

### Human-Readable Status Codes

Instead of returning raw numbers, DRF provides named constants that make code more readable:

```py
from rest_framework import status

# Instead of writing 200, 201, 404...
status.HTTP_200_OK           # 200
status.HTTP_201_CREATED      # 201
status.HTTP_400_BAD_REQUEST  # 400
status.HTTP_404_NOT_FOUND    # 404
```

## Ways to Write Views in DRF

DRF gives you three levels of abstraction for writing API views, from most manual to most automatic:

| Approach              | What it is                                   | Best for                              |
| --------------------- | -------------------------------------------- | ------------------------------------- |
| `@api_view` decorator | Function-based views (FBV)                   | Simple, one-off endpoints             |
| `APIView` class       | Class-based views (CBV)                      | More control, reusable logic          |
| Generic views         | Pre-built classes for common CRUD operations | Full CRUD endpoints with minimal code |

Each approach builds on the previous one. You will use all three depending on the situation. They are documented in the other files in this folder.

## Key Advantages of DRF

- Easy to integrate into any existing Django project
- Handles JSON parsing, serialization, and response formatting automatically
- Provides a browsable API interface (you can test your endpoints in the browser)
- Built-in authentication support (token-based, session, JWT via third-party)
- Built-in throttling (rate limiting) and permission classes
- Well-documented and widely used in production
