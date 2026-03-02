# Class-Based Views (APIView)

## What is a Class-Based View?

A class-based view (CBV) is a view written as a Python class instead of a plain function. Each HTTP method (GET, POST, PUT, DELETE) becomes its own method inside the class, instead of being handled with `if request.method == 'GET'` checks.

In DRF, you create a class-based view by inheriting from `APIView`.

## Why Use Class-Based Views?

As your API grows, function-based views can get cluttered — one function ends up handling GET, POST, PUT, and DELETE with multiple `if` branches. Class-based views split each method into its own clearly named method, making the code easier to read and maintain.

### Comparison

Function-based view:

```py
@api_view(['GET', 'POST'])
def courses(request):
    if request.method == 'GET':
        # get logic here
        pass
    elif request.method == 'POST':
        # post logic here
        pass
```

Class-based view (same logic, cleaner structure):

```py
class CourseList(APIView):
    def get(self, request):
        # get logic here
        pass

    def post(self, request):
        # post logic here
        pass
```

Each method name must exactly match the lowercase HTTP verb: `get`, `post`, `put`, `patch`, `delete`. DRF automatically routes incoming requests to the right method.

## Advantages of Class-Based Views

- Less code duplication — shared logic can go in the class or parent class
- Each HTTP method has its own clean, focused method
- Easy to extend — you can inherit from your own views to add or override behaviour
- Easier to apply permissions, authentication, and throttling at the class level

## Basic Example

```py
# views.py
from rest_framework.response import Response
from rest_framework import status
from rest_framework.views import APIView

class CourseList(APIView):
    def get(self, request):
        return Response({"message": "list of courses"}, status.HTTP_200_OK)

    def post(self, request):
        return Response({"message": "course created"}, status.HTTP_201_CREATED)
```

## Wiring Up URLs

Class-based views cannot be used directly in `urlpatterns` like a function. You must call `.as_view()` on the class, which converts it into a callable that Django's URL router understands.

```py
# CourseListAPI/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path("courses/", views.CourseList.as_view(), name="course-list"),
]
```

```py
# rest_api/urls.py  (main project URL file)
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path("api/", include("CourseListAPI.urls")),
]
```

With this setup, a request to `http://127.0.0.1:8000/api/courses/` will be routed to `CourseList`. A GET request calls `CourseList.get()`, a POST request calls `CourseList.post()`, and so on.

Note: It is good practice to include a trailing slash in your URL patterns. Some browsers and clients automatically add a trailing slash. Including it avoids redirect issues, especially when adding query parameters.

## Reading Query Parameters

Query parameters are the key-value pairs after the `?` in a URL (e.g., `?course=python`). You read them with `request.GET.get("key")`:

```py
class CourseList(APIView):
    def get(self, request):
        course = request.GET.get("course")   # returns None if not provided
        if course:
            return Response({"message": f"You requested: {course}"}, status.HTTP_200_OK)
        return Response({"message": "All courses"}, status.HTTP_200_OK)
```

Request: `http://127.0.0.1:8000/api/courses/?course=machine-learning`

## Reading Request Body Data (POST, PUT, PATCH)

When a client sends data in the request body (as JSON or form data), you read it with `request.data.get("key")`. This is DRF's version and works for any content type:

```py
class CourseList(APIView):
    def post(self, request):
        title = request.data.get("title")
        start_date = request.data.get("start_date")
        return Response({"message": f"{title} starts on {start_date}"}, status.HTTP_201_CREATED)
```

### request.data vs request.POST

| Aspect                      | `request.data` (DRF) | `request.POST` (Django) |
| --------------------------- | -------------------- | ----------------------- |
| Works with JSON             | Yes                  | No                      |
| Works with form data        | Yes                  | Yes                     |
| Returns None if key missing | Yes (with `.get()`)  | No — raises KeyError    |
| Recommended for APIs        | Yes                  | No                      |

Always use `request.data` in DRF views. It handles JSON (the standard for APIs), while `request.POST` only handles HTML form submissions.
