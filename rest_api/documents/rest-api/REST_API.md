## Application Programming Interface (API)

It is gate to backend data and it allows to access and modify these data.

## RESTful API

It is an architectural design to create the APIs for our project.
they provide an easy way to communicate with sever and access the data that powers our application.

an API is RESTful if it follow certain standards, here are the most important ones.

- It must have client-server architecture
- alway stateless
- should be cacheable
- it should be layered
- it should have a uniform interface

#### 1. client-server architecture

a client-server architecture contains a server to provide the resources and a client to consume them.

#### 2. stateless

it means server does not have any any info about state of the client who is making the call, So it cannot identify who is making the request or what their previously requested data was.
in fact, state is only save on client machine not on the server.
to help the server, more information should be included in an API.

#### 3. cacheable

the response should be savable on browser or server and it helps to reduce the server load by using the saved response instead of making a new request.

#### 4. layered

it should have different layers and those layers are removable any time.

the layers could be

```
|Client| --> |Firewall| --> |Load balancer| --> |web server| --> |database server|
```

#### 5. Uniform interface

it means the system should offer a uniform interface to access the resource, for instance, there should be unique `URL` for each resource.

```py
from django.shortcuts import render
from rest_framework.response import Response
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.views import APIView


# Create your views here.

@api_view(["POST","GET"])
def courses(request):
    return Response("list of courses provided by hooshmandlab",status=status.HTTP_200_OK)


class Courses(APIView):
    def get(self, request):
        instructor = request.GET.get("instructor")
        if instructor:
            return Response({"messages":"list of courses by "+instructor}, status.HTTP_200_OK)
        return Response({"messages":"list of courses"}, status.HTTP_200_OK)


    def post(self, request):
        title = request.data.get("title")
        return Response({"message":title + " created"}, status=status.HTTP_200_OK)


class Course(APIView):
    def get(self, request, pk):
        return Response({"message":"course with id = " + str(pk)})
    def put(self, request, pk):
        title = request.data.get("title")
        return Response({"message":title + " created"})
```

```py
class Courses(APIView):
    def get(self, request):
        courses = Course.objects.all()
        return Response(courses.values())

    def post(self, request):
        title = request.data.get("title")
        return Response({"message":title + " created"}, status=status.HTTP_200_OK)

```

### Tools

- `curl https://www.coursera.org/learn/apis/lecture/GfIq7/essential-tools-for-api-development` can be used to make get request from a terminal.
