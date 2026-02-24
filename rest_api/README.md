# Rest API Documentation

Welcome to the **Rest API Documentation** repository! This repository contains detailed documentation for building and managing RESTful APIs using Django Rest Framework (DRF) with authentication, serialization, and various advanced features like filtering, pagination, and caching.

This documentation is organized into several key sections to help developers build scalable and maintainable APIs for applications like online learning platforms, It covers topics such as authentication, search filtering, API serialization, validation, and much more.

### Response and URLS

- [Command](./documents/command.md)
- [HTTP-HTTPS](./documents/02-http-https/http_https.md)
- [Response-types](./documents/02-http-https/response_types.md)
- [URL](./documents/02-http-https/url.md)
- [Query-string-and-params](./documents/02-http-https/query_string_and_params.md)

#### RESTful API

- [RESTful-API](./documents/01-rest-api/readme.md)
- [Naming-conventions](./documents/01-rest-api/naming_conventions.md)
- [Rest-principles](./documents/01-rest-api/rest_principles.md)
- [Security-practices](./documents/01-rest-api/security.md)
- [Setup](./documents/setup.md)

#### Django Rest Framework

- [DRF](./documents/drf/readme.md)
- [API-view](./documents/drf/01_api_view.md)
- [Class-based-view](./documents/drf/02_class_based_view.md)
- [Generics-views](./documents/drf/03_generics.md)

#### Serialization

- [Serializers](./documents/serialization/serializers.md)
- [Generics-views](./documents/serialization/generics.md)
- [Deserialization](./documents/serialization/deserializer.md)
- [Sanitization](./documents/serialization/sanitization.md)
- [Model-creation](./documents/serialization/model_creation.md)
- [Validation](./documents/serialization/validation.md)
- [Renderers](./documents/serialization/renderers.md)
- [Renderers](./documents/serialization/model_creation.md)

#### Filter/Search

- [Filtering-searching](./documents/filter-search/filtering.md)
- [Filtering-class-based-views](./documents/filter-search/filtering_classbased_views.md)
- [Ordering](./documents/filter-search/ordering.md)
- [Caching](./documents/caching/readme.md)
- [Pagination](./documents/filter-search/pagination.md)

#### Authentication and securing APIs

- [API-restriction](./documents/authentication/auth.md)
- [Djoser](./documents/authentication/djoser.md)
- [JWT](./documents/authentication/jwt.md)
- [Throttling](./documents/authentication/throttling.md)
- [Token](./documents/authentication/token.md)

### Resources

- [http](https://httpbin.org/#/Status_codes)
- [status-code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

### Tools

- [insomnia](https://docs.insomnia.rest/)
- [postman](https://www.postman.com/)
- [psotman-docs](https://www.postman.com/postman/published-postman-templates/documentation/ae2ja6x/postman-echo?ctx=documentation)

## Getting Started

### Prerequisites

- Python 3.x
- Django
- Django Rest Framework (DRF)

### Installation

To get started with this repository, clone it to your local machine and follow the instructions in the `setup.md` document for detailed setup steps.

### Features

- **Authentication**: Covers JWT, Token-based authentication, and Djoser integration.
- **Filtering and Search**: Learn how to implement powerful filtering and searching mechanisms.
- **Serialization**: Detailed explanations of how to serialize and validate your API data.
- **Pagination and Throttling**: Manage large datasets efficiently with pagination and control API usage with throttling.

Feel free to explore each document in the repository for comprehensive guidelines and code examples to help you integrate these features into your own projects.

---

If you have any questions, suggestions, or issues, please open a pull request or an issue in the repository.
