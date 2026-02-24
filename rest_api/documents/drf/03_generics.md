# Generic Views

## What are Generic Views?

Generic views are pre-built class-based views that DRF provides for the most common API operations: listing resources, creating resources, retrieving a single resource, updating, and deleting.

With `APIView` (covered in `02_class_based_view.md`), you write the logic for each HTTP method yourself. With generic views, DRF already knows what a list endpoint or a create endpoint should do - you just tell it which model to use and which serializer to use, and it handles the rest automatically.

Think of it as going from writing everything by hand to filling in a form.

## APIView vs Generic Views

```py
# With APIView — you write all the logic yourself
class CourseList(APIView):
    def get(self, request):
        courses = Course.objects.all()
        serializer = CourseSerializer(courses, many=True)
        return Response(serializer.data)

    def post(self, request):
        serializer = CourseSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

```py
# With Generic Views — DRF handles the same logic for you
class CourseList(generics.ListCreateAPIView):
    queryset = Course.objects.all()
    serializer_class = CourseSerializer
```

Both do exactly the same thing. The generic view version is much shorter because all the repetitive logic (checking validity, saving, returning the right status code) is already built in.

## Available Generic View Classes

| Generic View Class             | HTTP Methods            | Purpose                                 |
| ------------------------------ | ----------------------- | --------------------------------------- |
| `CreateAPIView`                | POST                    | Create a new resource                   |
| `ListAPIView`                  | GET                     | Return a list of all resources          |
| `RetrieveAPIView`              | GET                     | Return a single resource by ID          |
| `DestroyAPIView`               | DELETE                  | Delete a single resource                |
| `UpdateAPIView`                | PUT, PATCH              | Replace or partially update a resource  |
| `ListCreateAPIView`            | GET, POST               | List all resources and create a new one |
| `RetrieveUpdateAPIView`        | GET, PUT, PATCH         | Retrieve and update a single resource   |
| `RetrieveDestroyAPIView`       | GET, DELETE             | Retrieve and delete a single resource   |
| `RetrieveUpdateDestroyAPIView` | GET, PUT, PATCH, DELETE | Full CRUD on a single resource          |

## What Every Generic View Needs

Every generic view requires two things at minimum:

1. `queryset` — the database query that retrieves the records (e.g., all courses)
2. `serializer_class` — the serializer that converts the model instances to/from JSON

```py
# views.py
from rest_framework import generics
from .models import Course
from .serializers import CourseSerializer

class CourseList(generics.ListCreateAPIView):
    queryset = Course.objects.all()
    serializer_class = CourseSerializer
```

That is all. DRF handles the GET (list) and POST (create) logic automatically.

## Serializer Example

The serializer tells DRF which fields to include when converting a model to JSON (and validating incoming JSON data):

```py
# serializers.py
from rest_framework import serializers
from .models import Course

class CourseSerializer(serializers.ModelSerializer):
    class Meta:
        model = Course
        fields = ["id", "course_name", "slug"]
```

`ModelSerializer` automatically generates fields based on your Django model, so you do not have to define each field by hand. You just list which fields to expose in `fields`.

## Choosing Which Generic View to Use

A common situation: you want an endpoint that lists all courses AND lets you create a new one.

**Option 1 — Combine two separate generic views:**

```py
from rest_framework import generics

class CoursesView(generics.CreateAPIView, generics.ListAPIView):
    queryset = Course.objects.all()
    serializer_class = CourseSerializer
```

**Option 2 — Use the combined view (simpler and preferred):**

```py
from rest_framework import generics

class CoursesView(generics.ListCreateAPIView):
    queryset = Course.objects.all()
    serializer_class = CourseSerializer
```

`ListCreateAPIView` already combines both — use Option 2 unless you have a specific reason not to.

## Wiring Up URLs

For endpoints that operate on the whole collection (list/create), no ID is needed in the URL. For endpoints that operate on a single item (retrieve/update/delete), an ID (or primary key) is required:

```py
# CourseListAPI/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path("courses/", views.CourseList.as_view()),         # list + create
    path("courses/<int:pk>/", views.CourseDetail.as_view()),  # retrieve + update + delete
]
```

`<int:pk>` is a URL parameter. `pk` stands for primary key — the unique ID of the record in the database. DRF's generic views automatically use `pk` to look up the specific object.

## Permission Classes

You can restrict who can access a generic view using `permission_classes`. For example, require authentication for all methods:

```py
from rest_framework.permissions import IsAuthenticated

class CourseList(generics.ListCreateAPIView):
    queryset = Course.objects.all()
    serializer_class = CourseSerializer
    permission_classes = [IsAuthenticated]
```

To apply permissions only to certain methods (e.g., allow GET without login but require authentication for POST), override `get_permissions()`:

```py
from rest_framework.permissions import IsAuthenticated

class CourseList(generics.ListCreateAPIView):
    queryset = Course.objects.all()
    serializer_class = CourseSerializer

    def get_permissions(self):
        # GET is open to anyone; POST requires authentication
        if self.request.method != 'GET':
            return [IsAuthenticated()]
        return []
```

`get_permissions()` is called on every request, so the permission check is dynamic — it depends on the method being used.

## Overriding Default Behaviour

Sometimes you need to change what a generic view does by default. You can override any method:

```py
class CourseList(generics.ListCreateAPIView):
    queryset = Course.objects.all()
    serializer_class = CourseSerializer

    def get(self, request, *args, **kwargs):
        # custom logic before the standard list response
        return Response({"message": "custom response"})
```

Use this sparingly — if you are overriding everything, you might as well use `APIView` directly.
