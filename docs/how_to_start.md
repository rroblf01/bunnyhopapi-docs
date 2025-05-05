# How to Start with the Bunnyhopapi HTTP Server

This document explains how the HTTP server implemented with **Bunnyhopapi** works and how to interact with it.

## Overview

The HTTP server uses the following key technologies and concepts:

- **Pydantic**: For data validation and serialization.
- **Bunnyhopapi**: For creating servers and routers.
- **Middlewares**: To handle additional logic before and after requests.
- **Async/Sync**: Use your asynchronous and synchronous methods to handle requests efficiently.
- **Swagger**: Automatic documentation generation for your endpoints.
- **CORS**: CORS configuration to allow requests from different origins.

## Creating the Server

To create a basic HTTP server, you can use the following code:

```python
from bunnyhopapi.server import Server

if __name__ == "__main__":
    server = Server()
    server.run()
```

With this code, the server will run on `localhost` on port `8000` by default. You can access it by opening your browser and navigating to `http://localhost:8000`.
You can stop the server by pressing `Ctrl + C` in the terminal where it is running.

## Server Configurations
You can customize the HTTP server using various configurations. Here are some of the most common configurations:
- **Host**: Change the host where the server runs. By default, it is 0.0.0.0 (all network interfaces).
- **Port**: Change the port where the server runs. By default, it is 8000.
- **CORS**: Configure CORS to allow requests from different origins.
- **Workers**: Change the number of workers to handle multiple simultaneous requests. By default, it is the number of processor cores.

## Configuration Example
```python
import os
from bunnyhopapi.server import Server

if __name__ == "__main__":
    server = Server(port=int(os.getenv("PORT", 8000)), host=os.getenv("HOST", "0.0.0.0"), cors=bool(os.getenv("CORS", False)))
    server.run(workers=os.getenv("WORKERS", 1))
```

## Creating Endpoints
### Class Mode
To create an endpoint, you must define a class that inherits from `Endpoint` and define methods with the corresponding middlewares to handle HTTP requests. Here's a basic example:

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
In this example, we have created a `/health` endpoint that responds to GET, POST, PUT, PATCH, and DELETE requests with a JSON message.

### Function Mode

You can also create endpoints using functions instead of classes. Here's a basic example:

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
In this example, we have created a `/health` endpoint that responds to GET, POST, PUT, PATCH, and DELETE requests with a JSON message.

## Path and Query Variables

To define path variables, the variable in the method must be typed as `PathParam`, and to define query variables, the variable in the method must be typed as `QueryParam`. Here's a basic example:

### Function Mode
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

### Class Mode
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

In this example, we have created a `/user` endpoint that responds to GET requests with a JSON message that includes the `user_id` and the `age` from the query.
Try accessing http://localhost:8000/docs, and you will see the automatically generated documentation for your endpoint.

If you need them to be optional, you can give them a default value.

## Typing
To define the type of parameters, you can use Python's type annotations. Bunnyhopapi uses Pydantic for data validation and serialization, which means you can define your parameters as standard Python data types (int, str, float, etc.) or as custom Pydantic models.

And to type the response, a dictionary is used, where the key is the status code, and the value is a dictionary with the response type.

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

Bunnyhopapi will automatically validate all data types, including path, query, and request body, and will return a 422 error if any of the types do not match the expected ones.

