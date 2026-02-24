# Rest API Documentation

Welcome to the **Rest API Documentation** repository! This repository contains detailed documentation for building and managing RESTful APIs using Django Rest Framework (DRF) with authentication, serialization, and various advanced features like filtering, pagination, and caching.

This documentation is organized into several key sections to help developers build scalable and maintainable APIs for applications like online learning platforms, It covers topics such as authentication, search filtering, API serialization, validation, and much more.

### Response and URLS

- [Command](./rest_api/documents/command.md)
- [HTTP-HTTPS](./rest_api/documents/02-http-https/http_https.md)
- [Response-types](./rest_api/documents/02-http-https/response_types.md)
- [URL](./rest_api/documents/02-http-https/url.md)
- [Query-string-and-params](./rest_api/documents/02-http-https/query_string_and_params.md)

#### RESTful API

- [RESTful-API](./rest_api/documents/01-rest-api/readme.md)
- [Naming-conventions](./rest_api/documents/01-rest-api/naming_conventions.md)
- [Rest-principles](./rest_api/documents/01-rest-api/rest_principles.md)
- [Security-practices](./rest_api/documents/01-rest-api/security.md)
- [Setup](./rest_api/documents/setup.md)

#### Django Rest Framework

- [DRF](./rest_api/documents/drf/readme.md)
- [API-view](./rest_api/documents/drf/01_api_view.md)
- [Class-based-view](./rest_api/documents/drf/02_class_based_view.md)
- [Generics-views](./rest_api/documents/drf/03_generics.md)

#### Serialization

- [Serializers](./rest_api/documents/serialization/serializers.md)
- [Generics-views](./rest_api/documents/serialization/generics.md)
- [Deserialization](./rest_api/documents/serialization/deserializer.md)
- [Sanitization](./rest_api/documents/serialization/sanitization.md)
- [Model-creation](./rest_api/documents/serialization/model_creation.md)
- [Validation](./rest_api/documents/serialization/validation.md)
- [Renderers](./rest_api/documents/serialization/renderers.md)
- [Renderers](./rest_api/documents/serialization/model_creation.md)

#### Filter/Search

- [Filtering-searching](./rest_api/documents/filter-search/filtering.md)
- [Filtering-class-based-views](./rest_api/documents/filter-search/filtering_classbased_views.md)
- [Ordering](./rest_api/documents/filter-search/ordering.md)
- [caching](./rest_api/documents/caching/readme.md)
- [caching-types](./rest_api/documents/caching/caching_types.md)
- [redis](./rest_api/documents/caching/redis.md)
- [redis-gcp](./rest_api/documents/caching/redis_gcp.md)

#### Authentication and securing APIs

- [API-restriction](./rest_api/documents/authentication/auth.md)
- [Djoser](./rest_api/documents/authentication/djoser.md)
- [JWT](./rest_api/documents/authentication/jwt.md)
- [Throttling](./rest_api/documents/authentication/throttling.md)
- [Token](./rest_api/documents/authentication/token.md)

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

If you have any questions, suggestions, or issues, please open a ful request or an issue in the repository.
