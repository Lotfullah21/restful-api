# Django Debug Toolbar

## What is it?

The Django Debug Toolbar is a configurable set of panels that appear in the browser while you are running your Django project in development mode. It shows you detailed information about what is happening behind the scenes for each request — things you cannot see just by looking at the API response.

It is a development-only tool. It is never shown in production.

## What Does it Show You?

Each panel in the toolbar shows a different category of information:

| Panel       | What it tells you                                                                                   |
| ----------- | --------------------------------------------------------------------------------------------------- |
| SQL Queries | Every database query that ran for this request, how long each took, and whether any were duplicated |
| Request     | All headers, cookies, session data, GET and POST parameters sent with the request                   |
| Headers     | HTTP request and response headers                                                                   |
| Settings    | All Django settings currently active                                                                |
| Templates   | Which templates were rendered and with what context                                                 |
| Cache       | Cache hits and misses                                                                               |
| Signals     | Django signals that were triggered                                                                  |
| Profiling   | How long each part of your code took to execute                                                     |

The most useful panel for API development is **SQL Queries**. It shows you if your view is accidentally making too many database queries (a common performance problem called the N+1 problem).

## Installation

### 1. Install the package

```sh
pip install django-debug-toolbar
```

### 2. Add to INSTALLED_APPS in settings.py

```py
# settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    # ... other apps
    'debug_toolbar',   # add this
]
```

### 3. Add the middleware in settings.py

Middleware is code that runs on every request and response, before and after your view. The toolbar's middleware intercepts responses and injects the toolbar HTML into them.

```py
# settings.py
MIDDLEWARE = [
    'debug_toolbar.middleware.DebugToolbarMiddleware',  # add this at the top
    'django.middleware.security.SecurityMiddleware',
    # ... other middleware
]
```

Place it as close to the top of the list as possible so it wraps around everything else.

### 4. Restrict it to your local machine

The toolbar only shows for IP addresses listed in `INTERNAL_IPS`. This prevents it from ever appearing to real users if it somehow ends up on a production server. `127.0.0.1` is your own machine (localhost):

```py
# settings.py
INTERNAL_IPS = [
    '127.0.0.1',
]
```

Optionally, you can configure it to start collapsed so it does not block your view:

```py
# settings.py
DEBUG_TOOLBAR_CONFIG = {
    'SHOW_COLLAPSED': True,   # toolbar starts collapsed, click to expand
}
```

### 5. Add the debug toolbar URL to your main urls.py

The toolbar needs its own URL route to serve its assets (CSS, JavaScript). This route should only exist when `DEBUG` is `True` (i.e., in development):

```py
# rest_api/urls.py  (main project URL file)
from django.contrib import admin
from django.urls import path, include
from django.conf import settings

urlpatterns = [
    path('admin/', admin.site.urls),
    path("api/", include("CourseListAPI.urls")),
]

if settings.DEBUG:
    import debug_toolbar
    urlpatterns += [path('__debug__/', include(debug_toolbar.urls))]
```

`settings.DEBUG` is `True` when you are running the development server and `False` in production. The `if` block ensures the toolbar route is never added in production.

## How to Use it

1. Run your development server: `python manage.py runserver`
2. Open any page in the browser that your Django app serves
3. The toolbar appears as a sidebar on the right side of the page
4. Click any panel title to expand it and see the details

Note: The toolbar only appears on responses that return HTML. It does not appear on raw JSON API responses. To inspect API requests, either use the browsable API (visit the endpoint in a browser with DRF installed) or look at the toolbar on an HTML page that triggers the API call.
