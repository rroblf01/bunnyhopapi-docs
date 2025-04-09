## Information
As part of the application setup, the router can be configured to use a different route prefix than the default. This is useful if you are running multiple services on the same host and want each to have its own route prefix.
## Example
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

## Explanation
In this example, we have created an API router with the prefix `/api` and a version router with the prefix `/v1`. Then, we included the API router within the version router. This means that the final route for the health endpoint will be `/v1/api/health`.
