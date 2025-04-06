# Installation and Usage of BunnyHopApi

---

## Installation

**Installation from PyPI**:

   You can install BunnyHopApi directly from the Python Package Index (PyPI) using `pip`. Open your terminal and run the following command:

   ```bash
   pip install bunnyhopapi
   ```

---

## Usage Example

Below is a basic example of how to set up and use BunnyHopApi to create a simple HTTP endpoint.

### Example: HTTP Endpoint

```python
from bunnyhopapi.server import Server
from bunnyhopapi.models import Endpoint


class HealthEndpoint(Endpoint):
    path = "/health"

    @Endpoint.GET()
    def get(self, headers):
        return 200, {"message": "GET /health"}


if __name__ == "__main__":
    server = Server()
    server.include_endpoint_class(HealthEndpoint)
    server.run()
```

## Code Explanation

1. **Module Import**:
   - We import `Server` to create the server.
   - We import `Endpoint` to define our endpoints.

2. **Endpoint Definition**:
   - We create a class `HealthEndpoint` that inherits from `Endpoint`.
   - We define the endpoint path with `path = "/health"`.
   - We implement a method with the `GET` decorator and make it return a status code and a JSON.

3. **Server Configuration**:
   - We create an instance of `Server`.
   - We add the `HealthEndpoint` to the server.

4. **Server Execution**:
   - We run the server by calling `server.run()`.

---

With these steps, you will have a basic server running that responds to HTTP requests at the `/health` endpoint.
By default, it will run with the host `localhost` and port `8000`. You can access it by opening your browser and navigating to `http://localhost:8000/health`.
You can also use the automatically generated Swagger documentation to explore and test your API at `http://localhost:8000/docs`.
Happy development with BunnyHopApi!
