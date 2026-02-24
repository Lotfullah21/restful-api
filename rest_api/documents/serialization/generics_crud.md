## Generics Crud operations

To handle creation of an object, three steps needs to be followed.

#### 1. Create an API view to receive the request

```py
# views.py
from .serializers import CourseSerializer
from rest_framework import generics
class CourseCreate(generics.CreateAPIView):
    queryset = Course.objects.all()
    serializer_class = CourseSerializer
```

#### 2. Create a serializer to handle the validation

```py
class CourseSerializer(serializers.ModelSerializer):
    modules = ModuleSerializer(read_only=True, many=True)
    subject = SubjectSerializer(read_only=True)
    class Meta:
        model = Course
        fields = ("id","title", "description","level","language","thumbnail",
                  "course_start_date","course_end_date","created_at",
                  "updated_at","is_published","subject","modules")
    read_only_fields = ['created_by']
```

#### 3. Create a url patter that maps the request to the api end point

```py

from django.urls import path
from .views import  CourseList, SingleCourse, CourseCreate, CourseUpdate, CourseDelete

urlpatterns = [
    path("courses/",CourseList.as_view(), name="course_list"),
    path("courses/<int:pk>",SingleCourse.as_view(), name="single_course"),
    path("courses/new/",CourseCreate.as_view(), name="course_create"),
    path("courses/<int:pk>/update/",CourseUpdate.as_view(), name="course_update"),
    path("courses/<int:pk>/delete/",CourseDelete.as_view(), name="course_delete"),

]

```

#### 4. Post the data to the url end point

based on the route, different views will be opened.
