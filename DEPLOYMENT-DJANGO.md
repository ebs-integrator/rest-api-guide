# Checklist for a Django deploy-ready project

1. In root should be a working Dockerfile customized for this project;

2. Created docker image should be optimized to have a minimum size, 300+ MB it's too much;

3. Application API should be served with gunicorn, uvicorn or daphne, no nginx, supervisor, or other layers

4. Avoid serving static files from API container, if it's impossible then use a nginx container to serve files from the
   shared directory

5. Running project should read environment variables for configuration, no mounted files like .env or settings.py;

```python
import environ

env = environ.Env(
    DEBUG=(bool, False),
    DEBUG_LEVEL=(str, 'INFO'),
    SEND_TO_SENTRY=(bool, False)
)

environ.Env.read_env(f"{BASE_DIR}/.env")

SECRET_KEY = env('SECRET_KEY')
DEBUG = env('DEBUG')
DEBUG_LEVEL = env('DEBUG_LEVEL')
```

6. Project should have valid migration files, with no conflicts or corruptions;

7. Project should have a health check endpoint on address GET /health or GET /common/health and return 200 response code
   if everything is ok;

8. If the project uses Redis only for cache, it should work without it, like an additional service, if the redis server is down then
   the project should not crash;

```python
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": CACHE_REDIS_LOCATION,
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
            "SOCKET_CONNECT_TIMEOUT": 0.25,
            "SOCKET_TIMEOUT": 0.25,
            "IGNORE_EXCEPTIONS": True,
        }
    }
}
```

9. Project should have clean warning list on Django Check `python manage.py check --deploy`

10. Project should start without database schema created if you have a select on app.ready method please catch the error
    of no table or no data to run `python manage.py migrate`;

11. Project should use the persistent connection on the production version,
    see https://docs.djangoproject.com/en/3.2/ref/databases/#persistent-database-connections

12. If the project need some initial data it should be created on migration call; see
    https://docs.djangoproject.com/en/3.2/ref/migration-operations/#django.db.migrations.operations.RunPython

13. On project startup check all dependencies if the connection is all right, for database, Redis, Rabbitmq, ElasticSearch,
    others API (by checking their health checks) and return human-readable message;

```python
from django.apps import AppConfig
from django.conf import settings


class CommonConfig(AppConfig):
    name = 'apps.common'

    def ready(self):

        # Check database connection
        from django.db import connections
        from django.db.utils import OperationalError
        try:
            with connections['default'].cursor() as cursor:
                cursor.close()
        except OperationalError as e:
            raise RuntimeError('No DATABASE connection, please check server availability: ' + str(e))

        # Check RabbitMQ connection
        from kombu import Connection
        try:
            with Connection(settings.CELERY_BROKER_URL).channel() as channel:
                channel.close()
        except Exception as e:
            raise RuntimeError('No RABBITMQ connection, please check server availability: ' + str(e))

        # Check ElasticSearch connection
        from elasticsearch import Elasticsearch
        es = Elasticsearch([settings.ELASTIC_SEARCH_URL], verify_certs=True)
        if not es.ping(request_timeout=1):
            raise RuntimeError('No ELASTICSEARCH connection, please check server availability')

        # Check Redis connection
        from django_redis import get_redis_connection
        from redis import RedisError
        try:
            with get_redis_connection() as c:
                c.keys('no-key')
        except RedisError as e:
            raise RuntimeError('No REDIS connection, please check server availability: ' + str(e))

```