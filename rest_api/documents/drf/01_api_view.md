# Function-Based Views with @api_view

## What is a Function-Based View?

In Django, a view is a Python function (or class) that receives an HTTP request and returns an HTTP response. A function-based view (FBV) is the simplest kind — it is just a regular Python function.

```py
# A plain Django view (no DRF)
from django.http import HttpResponse

def courses(request):
    return HttpResponse("list of courses")
```

The problem with plain Django views for APIs is that you have to manually handle JSON parsing, content negotiation, and error responses. DRF solves this with the `@api_view` decorator.

## What is a Decorator?

A decorator is a Python feature that wraps a function to add extra behavior without changing the function itself. You apply it with the `@` symbol directly above the function definition.

```py
@api_view(['GET'])   # this is the decorator
def courses(request):
    ...
```

When `courses` is called, the decorator runs first, adds DRF's request/response handling, checks the HTTP method, and then calls your function. You do not need to write any of that setup code yourself.

## @api_view

`@api_view` is DRF's decorator for function-based views. It does three things:

1. Restricts which HTTP methods the view accepts (e.g., only GET and POST)
2. Automatically returns `405 Method Not Allowed` if an unsupported method is used
3. Wraps the request in DRF's `Request` object (so `request.data` works for JSON)

### Basic Usage

```py
# views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status

@api_view(['GET'])
def courses(request):
    return Response("list of courses", status=status.HTTP_200_OK)
```

If you call this endpoint with POST, DRF automatically returns `405 Method Not Allowed` — you do not need to write that check yourself.

### Default Behaviour (No Method Specified)

If you pass no methods to the decorator, it defaults to GET only:

```py
@api_view()
def courses(request):
    return Response("list of courses", status=status.HTTP_200_OK)
```

### Handling Multiple Methods

You can handle more than one HTTP method in the same function by checking `request.method`:

```py
from rest_framework.decorators import api_view
from rest_framework.response import Response

@api_view(['GET', 'POST'])
def courses(request):
    if request.method == 'GET':
        return Response({"message": "Here is the list of courses"})

    elif request.method == 'POST':
        title = request.data.get("title")   # request.data works for JSON and form data
        return Response({"message": f"{title} was created"})
```

Note: `request.data` is DRF's way of reading the request body. It works for JSON, form data, and multipart. Django's `request.POST` only works for HTML form submissions.

## Advantages of Function-Based Views

- Simple and easy to read — the entire logic is in one function
- Great for quick, one-off endpoints that do not need reuse
- Easier to use with decorators (e.g., adding permissions or throttling with one line)
- Good starting point when learning DRF before moving to class-based views

## Limitations

As your API grows, function-based views can get long and repetitive. If you have 10 endpoints that all do similar things (list, create, update, delete), you end up copying a lot of code. That is where class-based views (covered in `02_class_based_view.md`) become useful.
