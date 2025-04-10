## Como funciona
Bunnyhopapi permite middleware a nivel de servidor, de router, de clase Endpoint o de endpoint de la clase Endpoint.

Los middleware siempre reciben 2 parámetros obligatorios:
- `headers`: los headers de la petición.
- `endpoint`: el endpoint u otro middelware que se va a ejecutar.

Y siempre se ejecuta desde el más general al más específico, y permite tener varios middlewares al mismo tiempo.

## Middleware a nivel de servidor
``python
from bunnyhopapi.server import Server

def auth_middleware(headers, endpoint, *args, **kwargs):
    logger.info("auth_middleware: Before to call the endpoint")
    if "Authorization" not in headers:
        return 401, {"message": "Unauthorized"}
    logger.info("auth_middleware: After to call the endpoint")
    return endpoint(headers=headers, *args, **kwargs)

if __name__ == "__main__":
    server = Server(middleware=auth_middleware)
    server.run()
```

En este ejemplo hemos añadido una authentication middleware a nivel de servidor. Si no se encuentra el header `Authorization`, se devuelve un error 401. Si se encuentra, se llama al endpoint.

## Middleware a nivel de router
``python
from bunnyhopapi.server import Server, Router


def auth_middleware(headers, endpoint, *args, **kwargs):
    logger.info("auth_middleware: Before to call the endpoint")
    if "Authorization" not in headers:
        return 401, {"message": "Unauthorized"}
    logger.info("auth_middleware: After to call the endpoint")
    return endpoint(headers=headers, *args, **kwargs)


if __name__ == "__main__":
    server = Server()
    auth_router = Router(middleware=auth_middleware)
    server.include_router(auth_router)
    server.run(workers=1)
```
En este ejemplo hemos añadido una authentication middleware a nivel de router y solo afecta a los routers y endpoints que se añadan a este router.

## Middleware a nivel de clase Endpoint
``python

class UserEndpoint(Endpoint):
    path: str = "/users"

    @Endpoint.MIDDLEWARE
    def db_middleware(self, endpoint, headers, *args, **kwargs):
        logger.info("db_middleware: Before to call the endpoint")
        db = Database()
        return endpoint(headers=headers, db=db, *args, **kwargs)

    @Endpoint.GET()
    def get(self, headers, db: Database, *args, **kwargs) -> {200: UserList}:
        users = db.get_users()
        return 200, {"users": users}
```
En este ejemplo hemos añadido una middleware a nivel de clase Endpoint. Esta middleware se ejecuta antes de llamar al endpoint y se le pasa el endpoint, los headers y el resto de parámetros. En este caso, se crea una conexión a la base de datos y se pasa al endpoint.

## Middleware a nivel de endpoint
``python


def db_middleware(endpoint, headers, *args, **kwargs):
    logger.info("db_middleware: Before to call the endpoint")
    db = Database()
    return endpoint(headers=headers, db=db, *args, **kwargs)

class UserEndpoint(Endpoint):
    path: str = "/users"

    @Endpoint.GET(middleware=db_middleware)
    def get(self, headers, db: Database, *args, **kwargs) -> {200: UserList}:
        users = db.get_users()
        return 200, {"users": users}
```
En este ejemplo hemos añadido una middleware a nivel de endpoint. Esta middleware se ejecuta antes de llamar al endpoint y se le pasa el endpoint, los headers y el resto de parámetros. Este middleware solo afectaría a este endpoint y no a los demás.

## Conclusión
Bunnyhopapi permite añadir middlewares a nivel de servidor, router, clase Endpoint o endpoint de la clase Endpoint. Esto permite tener un control total sobre el flujo de ejecución y la seguridad de la aplicación.
Y al permitir varios middlewares a la vez, se puede dar el caso que un middleware llame a otro middleware y así hasta que lo que hayamos definido.