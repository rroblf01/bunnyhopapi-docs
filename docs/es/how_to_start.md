# Cómo empezar con el servidor HTTP de Bunnyhopapi

Este documento explica cómo funciona el servidor HTTP implementado con **Bunnyhopapi** y cómo interactuar con él.

## Descripción general

El servidor HTTP utiliza las siguientes tecnologías y conceptos clave:

- **Pydantic**: Para la validación y serialización de datos.
- **Bunnyhopapi**: Para la creación de servidores y enrutadores.
- **Middlewares**: Para manejar lógica adicional antes y después de las solicitudes.
- **Async/Sync**: Usa tus métodos asíncronos y sincrónicos para manejar las solicitudes de manera eficiente.
- **Swagger**: Generación automática de documentación para tus endpoints.
- **CORS**: Configuración de CORS para permitir solicitudes desde diferentes orígenes.

## Creación del servidor

Para crear un servidor HTTP básico, puedes usar el siguiente código:

```python
from bunnyhopapi.server import Server

if __name__ == "__main__":
    server = Server()
    server.run()
```

Con este código, el servidor se ejecutará en `localhost` en el puerto `8000` por defecto. Puedes acceder a él abriendo tu navegador y navegando a `http://localhost:8000`.
Puedes detener el servidor presionando `Ctrl + C` en la terminal donde se está ejecutando.

## Configuraciones del servidor
Puedes personalizar el servidor HTTP utilizando varias configuraciones. Aquí hay algunas de las configuraciones más comunes:
- **Host**: Cambia el host en el que se ejecuta el servidor. Por defecto es el 0.0.0.0 (todas las interfaces de red).
- **Port**: Cambia el puerto en el que se ejecuta el servidor. Por defecto es el 8000.
- **CORS**: Configura CORS para permitir solicitudes desde diferentes orígenes.
- **Workers**: Cambia el número de trabajadores para manejar múltiples solicitudes simultáneamente. Por defecto es el número de núcleos del procesador.

## Ejemplo de configuración
```python
import os
from bunnyhopapi.server import Server

if __name__ == "__main__":
    server = Server(port=int(os.getenv("PORT", 8000)), host=os.getenv("HOST", "0.0.0.0"), cors=bool(os.getenv("CORS", False)))
    server.run(workers=os.getenv("WORKERS", 1))
```

## Crear Endpoints
### Modo clases
Para crear un endpoint, debes definir una clase que herede de `Endpoint` y definir los métodos con los middlewares correspondientes para manejar las solicitudes HTTP. Aquí hay un ejemplo básico:

```python
from bunnyhopapi.models import Endpoint
from bunnyhopapi.server import Server

class HealthEndpoint(Endpoint):
    path = "/health"

    @Endpoint.GET()
    def get(self, headers):
        return 200, {"message": "GET /health"}

    @Endpoint.POST()
    def post(self, headers):
        return 200, {"message": "POST /health"}

    @Endpoint.PUT()
    def put(self, headers):
        return 200, {"message": "PUT /health"}

    @Endpoint.PATCH()
    def patch(self, headers):
        return 200, {"message": "PATCH /health"}

    @Endpoint.DELETE()
    def delete(self, headers):
        return 200, {"message": "DELETE /health"}

if __name__ == "__main__":
    server = Server()
    server.include_endpoint_class(HealthEndpoint)
    server.run()
```
En este ejemplo, hemos creado un endpoint `/health` que responde a las solicitudes GET, POST, PUT, PATCH y DELETE con un mensaje JSON.

### Modo funciones

También puedes crear endpoints utilizando funciones en lugar de clases. Aquí hay un ejemplo básico:

```python
from bunnyhopapi.server import Server

server = Server()

@server.get('/health')
def get(headers):
    return 200, {"message": "GET /health"}

@server.post('/health')
def post(headers):
    return 200, {"message": "POST /health"}

@server.put('/health')
def put(headers):
    return 200, {"message": "PUT /health"}

@server.patch('/health')
def patch(headers):
    return 200, {"message": "PATCH /health"}

@server.delete('/health')
def delete(headers):
    return 200, {"message": "DELETE /health"}

if __name__ == "__main__":
    server.run()
```
En este ejemplo, hemos creado un endpoint `/health` que responde a las solicitudes GET, POST, PUT, PATCH y DELETE con un mensaje JSON.

## Variables de parámetro y Query

Para definir variables de parámetro es necesario que la variable que en el método que definas se tipe como PathParam, y para definir variables de query es necesario que la variable que en el método que definas se tipe como QueryParam. Aquí hay un ejemplo básico:

### Modo funciones
```python
from bunnyhopapi.models import PathParam, QueryParam
from bunnyhopapi.server import Server

server = Server()

@server.get("/user/<user_id>")
def get(user_id: PathParam[int], age: QueryParam[int], headers):
    return 200, {"message": f"GET /user/{user_id}?age={age}"}

if __name__ == "__main__":
    server.run()
```

### Modo clases
```python
from bunnyhopapi.models import Endpoint, PathParam, QueryParam
from bunnyhopapi.server import Server


class UserEndpoint(Endpoint):
    path = "/user"

    @Endpoint.GET()
    def get(self, user_id: PathParam[int], age: QueryParam[int], headers):
        return 200, {"message": f"GET /user/{user_id}?age={age}"}


if __name__ == "__main__":
    server = Server()
    server.include_endpoint_class(UserEndpoint)
    server.run()
```

En este ejemplo, hemos creado un endpoint `/user` que responde a las solicitudes GET con un mensaje JSON que incluye el `user_id` y el `age` de la consulta.
Prueba a acceder a http://localhost:8000/docs y verás la documentación generada automáticamente para tu endpoint.

Si necesitas que sean opcionales, puedes darles un valor por defecto.


## Tipado 
Para definir el tipo de los parámetros, puedes usar las anotaciones de tipo de Python. Bunnyhopapi utiliza Pydantic para la validación y serialización de datos, lo que significa que puedes definir tus parámetros como tipos de datos estándar de Python (int, str, float, etc.) o como modelos Pydantic personalizados.

Y para tipar la respuesta, se usa un diccionario, donde la clave es el status code y el valor es un diccionario con el tipo de respuesta.

```python
from bunnyhopapi.models import Endpoint
from pydantic import BaseModel

class Message(BaseModel):
    message: str

class UserInput(BaseModel):
    name: str
    age: int

class UserOutput(BaseModel):
    id: str
    name: str
    age: int

class UserEndpoint(Endpoint):
    path = "/user"

    @Endpoint.POST()
    def post(self, user_input: UserInput, headers) -> {200: UserOutput}:
        user_output = UserOutput(id="123", name=user_input.name, age=user_input.age)
        return 200, user_output
```

Bunnyhopapi validará automáticamente todos los tipos de datos, tanto de parámetro, query y cuerpo de la solicitud, y devolverá un error 422 si alguno de los tipos no coincide con lo esperado.

