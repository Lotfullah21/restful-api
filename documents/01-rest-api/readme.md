## Application Programming Interface (API)

An API (Application Programming Interface) is a gate (entry point) to backend data. It allows clients (like a browser or mobile app) to access and modify data stored on a server. Think of it like a waiter in a restaurant: you (the client) give the waiter (the API) your order, and the waiter brings back food from the kitchen (the server).

## RESTful API

REST stands for Representational State Transfer. It is an architectural style (a set of rules and conventions) used to design APIs. RESTful APIs provide an easy, standardized way to communicate with a server and access the data that powers an application.

An API is considered RESTful if it follows certain standards. Here are the most important ones:

- It must have a client-server architecture
- It must always be stateless
- It should be cacheable
- It should be layered
- It should have a uniform interface

#### 1. Client-Server Architecture

This means the system is split into two separate parts:

- **Client**: The part that makes requests (e.g., a browser, mobile app, or frontend).
- **Server**: The part that holds the data and responds to requests (e.g., a Django backend).

They communicate over a network but are independent of each other. This separation means you can update the frontend without touching the backend, and vice versa.

#### 2. Stateless

Stateless means the server does not remember anything about the client between requests. Each request must include all the information the server needs to process it (e.g., who you are, what you want).

The server does not store any session or user information. That state lives only on the client side. This makes the server simpler and easier to scale.

Example: If you are logged in, your token (proof of identity) must be sent with every request. The server does not remember your previous request.

#### 3. Cacheable

Caching means saving a copy of a response so it can be reused later without making another request. For example, if you ask for a list of products and it rarely changes, the browser or server can store (cache) that response and serve it faster next time.

This reduces the load on the server and makes the API faster for the client.

#### 4. Layered

A layered system means the architecture can have multiple intermediate layers between the client and the server. Each layer only knows about the layer directly next to it, not the whole chain.

Common layers include:

```
|Client| --> |Firewall| --> |Load Balancer| --> |Web Server| --> |Database Server|
```

- **Firewall**: A security system that monitors and controls incoming and outgoing network traffic based on predefined rules. It acts as a barrier between trusted internal networks and untrusted external ones (like the internet). It blocks unauthorized access while allowing legitimate traffic through.
- **Load Balancer**: Distributes incoming requests across multiple servers so no single server gets overwhelmed. This improves performance and availability.
- **Web Server**: Handles the HTTP request and runs your application code (e.g., Django).
- **Database Server**: Stores and retrieves the data your application needs.

Each layer can be added, removed, or replaced independently without affecting the others.

#### 5. Uniform Interface

This means every resource in the system is accessed in a consistent, predictable way. There should be a unique URL for each resource, and the same HTTP methods (GET, POST, PUT, DELETE) should be used the same way everywhere.

Example: To get a list of courses, you always use `GET /courses`. To create one, you always use `POST /courses`. This consistency makes the API easy to understand and use.

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

- `curl` is a command-line tool used to make HTTP requests directly from a terminal. For example, `curl https://example.com/api/courses` sends a GET request to that URL and prints the response. It is useful for quickly testing API endpoints without needing a browser or GUI tool.
