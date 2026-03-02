## Generics

Generic views are another way of writing class-based views to write a fully functional CRUD API.

There are several generic views that offer a particular functionality, like displaying resources or creating a new resource and so on. All you must do is extend these generic views to make the API endpoints work.

```py
from rest_framework import generics
```

## Generic View Classes in Django REST Framework

| **Generic View Class**           | **Supported Methods**           | **Purpose**                                                   |
| -------------------------------- | ------------------------------- | ------------------------------------------------------------- |
| **CreateAPIView**                | `POST`                          | Create a new resource.                                        |
| **ListAPIView**                  | `GET`                           | Display a collection of resources.                            |
| **RetrieveAPIView**              | `GET`                           | Display a single resource.                                    |
| **DestroyAPIView**               | `DELETE`                        | Delete a single resource.                                     |
| **UpdateAPIView**                | `PUT`, `PATCH`                  | Replace or partially update a single resource.                |
| **ListCreateAPIView**            | `GET`, `POST`                   | Display a collection of resources and create a new resource.  |
| **RetrieveUpdateAPIView**        | `GET`, `PUT`, `PATCH`           | Display a single resource and replace or partially update it. |
| **RetrieveDestroyAPIView**       | `GET`, `DELETE`                 | Display a single resource and delete it.                      |
| **RetrieveUpdateDestroyAPIView** | `GET`, `PUT`, `PATCH`, `DELETE` | Display, replace or update, and delete a single resource.     |

For instance, we want our API end point to be capable of both displaying and posting a resource, what class to be used.

## Option-1

Use `CreateAPIView` for resource creation and `ListAPIView` for getting the resource.

```py
from rest_framework import generics
class CoursesView(generics.CreateAPIView, generics.ListAPIView):


```

## Option-2

Use `ListCreateAPIView` to do the both

```py
from rest_framework import generics
class CoursesView(generics.ListCreateAPIView):


```

theses generics classes needs two things:

1. `queryset`: to retrieve all the records
2. `serializer_class`: to display and save the records.

that's all, pass theses two to the class and everything is managed by the generics.

```py
# views.py
from rest_framework import generics
from .models import Course
from .serializers import CourseSerializer
class CoursesView(generics.ListCreateAPIView):
    queryset = Course.objects.all()
    serializer_class = CourseSerializer
```

```py
# serializers.py
from rest_framework import serializersc
from .models import Course
class CourseSerializer(serializers.ModelSerializer):
    class Meta:
        model = Course
        fields = ["id","course_name","slug"]
```

## Crucial

### Permission classes

To limit the accessibility of the resource, we need to use permission classes on what and who can and to do so, we have two options:

1. apply the permission classes on all methods like `GET, POST, PUT, DELETE`
2. apply the permission to certain kind of methods, for instance allowing only `GET` method. to go with this option, we need to override the get_permission method.

```py
def get_permissions(self):
        permission_classes = []
        if self.request.method != 'GET':
            permission_classes = [IsAuthenticated]
           
        return [permission() for permission in permission_classes]
```

### Override default behavior

We can override the automatic behavior of default methods.

```py
class CourseView(generics.ListCreateAPIView):
    queryset = Course.objects.all()
    serializer_class = CourseSerializer
    def get(self, request, *args, **kwargs):
        return Response(‘new response’)
```

## Single Item

### Retrieve View Usage:

The RetrieveAPIView is designed to retrieve a single object based on the pk (or other lookup field) provided in the request. The framework automatically filters the queryset based on the primary key or the lookup field provided in the URL.

```py
class SingleCourse(generics.RetrieveAPIView):
    queryset = Course.objects.all()
    serializer_class = CourseSerializer
```

Define a url pattern

```ruby
from django.urls import path
from .views import SingleCourse
urlpatterns = [
    path("courses/<int:pk>",SingleCourse.as_view(), name="single_course"),
]
```

## Posting data

When posting data, generally there are two scenarios.

1. straight forward, no relationship and all the fields match with what is expected
2. when we have foreign relationships and we want to handel them explicitly, to do this, we need to override the default create method in `serializers.py`

```py
from rest_framework import serializers
from .models import Course, CourseCategory


class CategorySerializer(serializers.ModelSerializer):
    class Meta:
        model = CourseCategory
        fields=["id","category_name"]

class CourseSerializer(serializers.ModelSerializer):
    category_id = serializers.PrimaryKeyRelatedField(queryset = CourseCategory.objects.all(), write_only=True)
    category = CategorySerializer(source="category_id", read_only=True)
    class Meta:
        model = Course
        fields = ["id","course_name","slug","category_id","category"]

    def create(self, validated_data):
        category_id = validated_data.pop("category_id")
        course = Course.objects.create(category_id=category_id, **validated_data)
        return course

```

To do the extra work and send a custom message after successful creation, override the create method in `views.py`.
