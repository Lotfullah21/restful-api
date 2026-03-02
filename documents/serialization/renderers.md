# Renderers in Django REST Framework (DRF)

Renderers are responsible for formatting the output of an API response in Django REST Framework. They allow APIs to support different response formats based on client needs.

## Built-in Renderers

DRF comes with the following built-in renderers:

- **`JSONRenderer`**: Displays data in JSON format (default for APIs).
- **`BrowsableAPIRenderer`**: Displays data in a user-friendly, browsable HTML format (useful for debugging).

## Third-Party Renderers

You can also use third-party renderers for additional formats:

- **`XMLRenderer`**: Displays data in XML format.
- **`YAMLRenderer`**: Displays data in YAML format.
- **`JSONPRenderer`**: Displays JSON data wrapped in a JavaScript function (JSONP format).

By default, DRF uses `JSONRenderer` and `BrowsableAPIRenderer`.

---

## Configuring Renderers

To customize the available renderers, update the `DEFAULT_RENDERER_CLASSES` setting in `settings.py`:

```python
REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer',
        'rest_framework.renderers.BrowsableAPIRenderer'
    ]
}
```

as a client, we need to add in the header in which format, we want our data to be in.

### Specifying Response Format

Clients can specify the desired response format by including an Accept Header in their API request. Here are some common options:

```table
| Accept Header                | Description                                       |
|------------------------------|---------------------------------------------------|
| `Accept: application/json`    | Expect JSON response.                             |
| `Accept: application/xml`     | Expect XML response.                              |
| `Accept: text/xml`            | Expect XML response (alternative format).        |
| `Accept: text/html`           | Expect HTML response (Browsable API).             |
| `Accept: text/plain`          | Expect plain text response.                       |
| `Accept: application/octet-stream` | Expect binary data response.              |

```

to add another support, let's say `XML`, first we need to install it,

```sh
pip install djangorestframework-xml
```

```py
#settings.py
REST_FRAMEWORK= {
    'DEFAULT_RENDERER_CLASSES':[
        'rest_framework.renderers.JSONRenderer',
        'rest_framework.renderers.BrowsableAPIRenderer',
        'rest_framework.renderers.BrowsableAPIRenderer'
    ]
}
```

### Example Usage

If a client wants the response in XML format, they can include the following header in their request:

```sh
Accept: application/xml
```

The server will then use the specified renderer to format the response accordingly.

## Best Practices

- Use JSONRenderer as the default renderer for most APIs, as JSON is widely supported and lightweight.
- Include BrowsableAPIRenderer during development for easier debugging and testing.
- Restrict renderers to only those formats required by your application to reduce overhead.
