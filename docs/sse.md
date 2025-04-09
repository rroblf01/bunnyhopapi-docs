## Explanation
To perform SSE (Server-Sent Events) in BunnyHopAPI, you can use the `@Endpoint.SSE()` decorator on your endpoint. This allows you to send events to the client continuously over an HTTP connection.
## Example
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

## Explanation
In this example, we have created an SSE endpoint at the `/sse/events` route. Inside the `get` method, we continuously generate events using `yield`. Each event is sent to the client with a 1.5-second interval between them. The client can listen to these events and process them as they arrive.