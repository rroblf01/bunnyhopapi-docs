## How it works
Bunnyhopapi allows middleware at the server level, router level, Endpoint class level, or endpoint level within the Endpoint class.

Middleware always receives 2 mandatory parameters:
- `headers`: the request headers.
- `endpoint`: the endpoint or other middleware to be executed.

And it always runs from the most general to the most specific, and allows you to have multiple middlewares at the same time.

## Server-level middleware
```python
from bunnyhopapi.server import Server

def auth_middleware(headers, endpoint, *args, **kwargs):
    logger.info("auth_middleware: Before calling the endpoint")
    if "Authorization" not in headers:
        return 401, {"message": "Unauthorized"}
    logger.info("auth_middleware: After calling the endpoint")
    return endpoint(headers=headers, *args, **kwargs)

if __name__ == "__main__":
    server = Server(middleware=auth_middleware)
    server.run()
```

In this example, we have added an authentication middleware at the server level. If the `Authorization` header is not found, a 401 error is returned. If it is found, the endpoint is called.

## Router-level middleware
```python
from bunnyhopapi.server import Server, Router


def auth_middleware(headers, endpoint, *args, **kwargs):
    logger.info("auth_middleware: Before calling the endpoint")
    if "Authorization" not in headers:
        return 401, {"message": "Unauthorized"}
    logger.info("auth_middleware: After calling the endpoint")
    return endpoint(headers=headers, *args, **kwargs)


if __name__ == "__main__":
    server = Server()
    auth_router = Router(middleware=auth_middleware)
    server.include_router(auth_router)
    server.run(workers=1)
```
In this example, we have added an authentication middleware at the router level, and it only affects the routers and endpoints added to this router.

## Endpoint class-level middleware
```python

class UserEndpoint(Endpoint):
    path: str = "/users"

    @Endpoint.MIDDLEWARE()
    def db_middleware(self, endpoint, headers, *args, **kwargs):
        logger.info("db_middleware: Before calling the endpoint")
        db = Database()
        return endpoint(headers=headers, db=db, *args, **kwargs)

    @Endpoint.GET()
    def get(self, headers, db: Database, *args, **kwargs) -> {200: UserList}:
        users = db.get_users()
        return 200, {"users": users}
```
In this example, we have added middleware at the Endpoint class level. This middleware runs before calling the endpoint and is passed the endpoint, headers, and other parameters. In this case, a database connection is created and passed to the endpoint.

## Endpoint-level middleware
```python


def db_middleware(endpoint, headers, *args, **kwargs):
    logger.info("db_middleware: Before calling the endpoint")
    db = Database()
    return endpoint(headers=headers, db=db, *args, **kwargs)

class UserEndpoint(Endpoint):
    path: str = "/users"

    @Endpoint.GET(middleware=db_middleware)
    def get(self, headers, db: Database, *args, **kwargs) -> {200: UserList}:
        users = db.get_users()
        return 200, {"users": users}
```
In this example, we have added middleware at the endpoint level. This middleware runs before calling the endpoint and is passed the endpoint, headers, and other parameters. This middleware would only affect this endpoint and not others.

## Conclusion
Bunnyhopapi allows you to add middleware at the server, router, Endpoint class, or endpoint level of the Endpoint class. This allows for complete control over the execution flow and security of the application.
And by allowing multiple middleware at the same time, one middleware can call another middleware, and so on, until the definition is reached.