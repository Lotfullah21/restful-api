## Redis

### Use Cases for Redis:

Query caching
Session caching (redis as session store)
Full-page caching for expensive views
Storing serialized objects, such as user profile data or API responses

it's ideal for session storage, query caching, and page caching. Since Redis is an in-memory data store, it will provide fast access to frequently used data, reducing database load.

- Install

```sh
pip install django-redis
```

- update settings.py

```py
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        },
    }
}
```

Why use 'CLIENT_CLASS': 'django_redis.client.DefaultClient'?
DefaultClient: This is the default class that django-redis uses to communicate with the Redis server. It's a well-tested and efficient client that works for most caching scenarios.
Thread-Safety: The default client is thread-safe, meaning it can be used in multi-threaded environments without issues.

### redis://: This specifies the protocol or scheme to use, which is redis://. It's the standard scheme for connecting to a Redis server.

127.0.0.1: This is the host (IP address) of the Redis server. In this case, 127.0.0.1 refers to localhost, which means the Redis server is running on the same machine as your Django application. If your Redis server is on a different machine, you would replace this with the server's IP address or hostname.

6379: This is the port where the Redis server is listening for incoming connections. By default, Redis uses port 6379, so unless you've changed the Redis configuration, this is the default port. If your Redis server uses a different port, you would change this value accordingly.

/1: This specifies the database number to use within the Redis server. Redis supports multiple databases (by default, Redis has 16 databases, indexed from 0 to 15). The /1 means you're using database 1. Redis databases are simple namespaces, and they are isolated from each other, so data in database 0 will not interfere with data in database 1, and so on. The default database is 0, but you can choose another database if needed.

## Crucial

Firebase Storage automatically uses Google Cloud CDN to cache and serve your media files globally.
Files uploaded to Firebase Storage will be distributed across Google's edge servers.

## Redis in GCP
