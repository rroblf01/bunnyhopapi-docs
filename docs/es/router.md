## Información
Como parte de la configuración de la aplicación, el enrutador se puede configurar para que use un prefijo de ruta diferente al predeterminado. Esto es útil si está ejecutando varios servicios en el mismo host y desea que cada uno tenga su propio prefijo de ruta.
## Ejemplo
```python
from bunnyhopapi.server import Server, Router
from bunnyhopapi.models import Endpoint


class HealthEndpoint(Endpoint):
    path = "/health"

    @Endpoint.GET()
    def get(self, headers):
        return 200, {"message": "GET /health"}


if __name__ == "__main__":
    server = Server()
    api_router = Router(prefix="/api")
    version_router = Router(prefix="/v1")

    version_router.include_router(api_router)

    server.include_router(version_router)

    server.run()
```

## Explicación
En este ejemplo, hemos creado un enrutador de API con el prefijo `/api` y un enrutador de versión con el prefijo `/v1`. Luego, incluimos el enrutador de API dentro del enrutador de versión. Esto significa que la ruta final para el punto final de salud será `/v1/api/health`.
