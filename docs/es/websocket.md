## Explicación

Para realizar WebSocket en BunnyHopAPI, debes crear un método llamado ws en tu endpoint. Este método se ejecutará cuando un cliente se conecte al WebSocket. 

## Ejemplo

```python
from bunnyhopapi.server import Server
from bunnyhopapi.models import Endpoint


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

## Explicación
En este ejemplo, hemos creado un endpoint WebSocket en la ruta `/ws/chat`. Dentro del método `ws`, recibimos el mensaje del cliente y lo procesamos. En este caso, simplemente enviamos un mensaje de vuelta al cliente cada 0.2 segundos. También hemos implementado los métodos `connection` y `disconnect` para manejar la conexión y desconexión del cliente.

En el método `connection` retornarás un booleano que indica si la conexión es válida o no. Si es válida, el cliente podrá conectarse al WebSocket. En el método `disconnect`, puedes realizar cualquier acción necesaria cuando un cliente se desconecta.