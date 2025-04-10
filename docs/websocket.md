## Explanation

To implement WebSocket in BunnyHopAPI, you need to create a method called `ws` in your endpoint. This method will execute when a client connects to the WebSocket.

## Example

```python
from bunnyhopapi.server import Server
from bunnyhopapi.models import Endpoint
import logging

logger = logging.getLogger(__name__)


class WSEndpoint(Endpoint):
    path = "/ws/chat"

    async def connection(self, headers):
        logger.info("Client connected")
        logger.info(f"Headers: {headers}")

        return True

    async def disconnect(self, connection_id: str, headers):
        logger.info(f"Client {connection_id} disconnected")

    async def ws(self, connection_id: str, message, headers):
        logger.info(f"Received message from {connection_id}: {message}")
        for i in range(10):
            yield f"event: message\ndata: {i}\n\n"
            await asyncio.sleep(0.2)


if __name__ == "__main__":
    server = Server()
    server.include_router(WSEndpoint)
    server.run()
```

## Explanation
In this example, we have created a WebSocket endpoint at the path `/ws/chat`. Inside the `ws` method, we receive the client's message and process it. In this case, we simply send a message back to the client every 0.2 seconds. We have also implemented the `connection` and `disconnect` methods to handle the client's connection and disconnection.

In the `connection` method, you will return a boolean indicating whether the connection is valid or not. If it is valid, the client will be able to connect to the WebSocket. In the `disconnect` method, you can perform any necessary actions when a client disconnects.