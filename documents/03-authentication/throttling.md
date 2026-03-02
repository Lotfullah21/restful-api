# API Throttling in DRF

Throttling limits request rate to prevent abuse and protect backend resources.

DRF throttling is not a security boundary by itself, but it is an effective rate-control layer.

## Global throttling setup

```py
# settings.py
REST_FRAMEWORK = {
    "DEFAULT_THROTTLE_CLASSES": [
        "rest_framework.throttling.AnonRateThrottle",
        "rest_framework.throttling.UserRateThrottle",
    ],
    "DEFAULT_THROTTLE_RATES": {
        "anon": "20/minute",
        "user": "200/minute",
    },
}
```

- `anon` applies when request is unauthenticated
- `user` applies when request is authenticated

## Function-based view examples

```py
# views.py
from rest_framework.decorators import api_view, permission_classes, throttle_classes
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.throttling import AnonRateThrottle, UserRateThrottle

@api_view(["GET"])
@throttle_classes([AnonRateThrottle])
def public_ping(request):
    return Response({"message": "public endpoint"})

@api_view(["GET"])
@permission_classes([IsAuthenticated])
@throttle_classes([UserRateThrottle])
def private_ping(request):
    return Response({"message": "authenticated endpoint"})
```

## Custom per-view throttle (scoped)

Create custom throttle class:

```py
# throttles.py
from rest_framework.throttling import UserRateThrottle

class FiveCallsPerMinute(UserRateThrottle):
    scope = "five_per_minute"
```

Add scope rate in settings:

```py
REST_FRAMEWORK["DEFAULT_THROTTLE_RATES"]["five_per_minute"] = "5/minute"
```

Use in a view:

```py
from .throttles import FiveCallsPerMinute

@api_view(["GET"])
@permission_classes([IsAuthenticated])
@throttle_classes([FiveCallsPerMinute])
def expensive_endpoint(request):
    return Response({"message": "limited endpoint"})
```

## Class-based view example

```py
from rest_framework.views import APIView
from rest_framework.throttling import AnonRateThrottle, UserRateThrottle
from rest_framework.response import Response

class CoursesView(APIView):
    throttle_classes = [AnonRateThrottle, UserRateThrottle]

    def get(self, request):
        return Response({"message": "courses"})
```

## Common mistakes

- Defining rates in settings but forgetting to attach throttle classes
- Assuming throttling replaces authentication/authorization
- Using very strict rates in development and thinking endpoints are broken
