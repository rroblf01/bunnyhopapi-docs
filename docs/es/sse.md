## Explicación
Para realizar SSE (Server-Sent Events) en BunnyHopAPI, puedes usar el decorador `@Endpoint.SSE()` en tu punto final. Esto te permite enviar eventos al cliente de forma continua a través de una conexión HTTP.
## Ejemplo
```python
from bunnyhopapi.server import Server
from bunnyhopapi.models import Endpoint
import asyncio


class SseEndpoint(Endpoint):
    path = "/sse/events"

    @Endpoint.GET(content_type=Server.CONTENT_TYPE_SSE)
    async def get(self, headers):
        events = ["start", "progress", "complete"]
        for event in events:
            yield f"event: {event}\ndata: Processing {event}\n\n"
            await asyncio.sleep(1.5)
        yield "event: end\ndata: Processing complete\n\n"


if __name__ == "__main__":
    server = Server()
    server.include_router(SseEndpoint)
    server.run()
```

## Explicación
En este ejemplo, hemos creado un punto final SSE en la ruta `/sse/events`. Dentro del método `get`, generamos eventos de forma continua utilizando `yield`. Cada evento se envía al cliente con un intervalo de 1.5 segundos entre ellos. El cliente puede escuchar estos eventos y procesarlos a medida que llegan.