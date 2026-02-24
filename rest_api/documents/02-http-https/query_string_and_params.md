# Query String and Query Parameters

When you see a URL like:

`/courses?category=backend&page=2&sort=name`

you are looking at query data.

## Core Definitions

- Query string: the full part after `?`
- Query parameter: one `key=value` pair inside the query string

From the example above:

- Query string: `category=backend&page=2&sort=name`
- Query parameters:
  - `category=backend`
  - `page=2`
  - `sort=name`

## Why They Exist

Query parameters let the client change how data is returned without changing the core resource path.

- `/courses` identifies the resource collection
- `?category=backend&page=2` adds options for filtering and pagination

## Common API Examples

```http
GET /courses
GET /courses?page=2&limit=10
GET /courses?category=backend
GET /courses?sort=created_at
GET /courses?category=backend&sort=name&page=1
GET /courses?search=django%20rest
```

## Query String vs Path Parameter

```http
GET /courses/42
```

- `42` is a path parameter (resource identity)

```http
GET /courses/42?include=lessons
```

- `42` is still a path parameter
- `include=lessons` is a query parameter

## Repeated Parameters

Some APIs support repeated keys:

```http
GET /courses?tag=python&tag=django
```

Possible meaning: return courses that match multiple tags.

## URL Encoding

Some values must be encoded:

- Space -> `%20` (or `+` in many query encoders)
- `&` and `=` are reserved in query strings

Example:

- Raw value: `machine learning`
- Encoded: `q=machine%20learning`

## Django and DRF Access

```py
# Django
def course_list(request):
    category = request.GET.get("category")
    page = request.GET.get("page", "1")
    return JsonResponse({"category": category, "page": page})
```

```py
# DRF
class CourseListView(APIView):
    def get(self, request):
        sort = request.query_params.get("sort", "name")
        return Response({"sort": sort})
```

## Quick Summary

- Use path parameters to identify resources.
- Use query parameters to filter, sort, search, and paginate.
- Query string = whole segment after `?`.
- Query parameter = each single `key=value` pair.
