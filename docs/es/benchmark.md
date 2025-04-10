# Comparativa de Rendimiento: Bunnyhopapi vs FastAPI vs Django

A continuación, se presenta una comparación de rendimiento entre **Bunnyhopapi**, **FastAPI** y **Django** utilizando el siguiente código de ejemplo para cada framework.

---

## Código de Bunnyhopapi

```python
from bunnyhopapi.server import Server
from bunnyhopapi.models import Endpoint


class HealthEndpoint(Endpoint):
    path = "/health"

    @Endpoint.GET()
    def get(self, headers):
        return 200, {"Hello": "World"}


if __name__ == "__main__":
    server = Server()
    server.include_endpoint_class(HealthEndpoint)
    server.run(workers=1)
```

### Benchmark de Bunnyhopapi

```bash
wrk -t12 -c400 -d30s http://127.0.0.1:8000/health

Running 30s test @ http://127.0.0.1:8000/health
  12 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    17.83ms   91.31ms   1.79s    98.66%
    Req/Sec     0.96k   587.57     2.56k    64.54%
  341722 requests in 30.08s, 37.80MB read
Requests/sec:  11358.95
Transfer/sec:      1.26MB
```

---

## Código de FastAPI

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}
```

### Benchmark de FastAPI

```bash
fastapi run main.py &
wrk -t12 -c400 -d30s http://127.0.0.1:8000/health

Running 30s test @ http://127.0.0.1:8000/health
  12 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   198.46ms  114.91ms   2.00s    95.19%
    Req/Sec   175.68     47.44     0.88k    88.62%
  61903 requests in 30.05s, 8.38MB read
Requests/sec:   2059.83
Transfer/sec:    285.64KB
```

---

## Código de Django

```python
from django.contrib import admin
from django.urls import path

from django.views import View
from django.http import JsonResponse

class MyView(View):

    def get(self, request, *args, **kwargs):
        return JsonResponse({'Hello': 'World!'})

urlpatterns = [
    path('admin/', admin.site.urls),
    path('health', MyView.as_view(), name='my-view'),
]
```

### Benchmark de Django

```bash
gunicorn mydjangoapp.wsgi &
wrk -t12 -c400 -d30s --timeout=1m http://127.0.0.1:8000/health

Running 30s test @ http://127.0.0.1:8000/health
  12 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   116.12ms    5.01ms 132.72ms   97.85%
    Req/Sec   284.80     49.70   560.00     80.69%
  102274 requests in 30.09s, 28.38MB read
Requests/sec:   3398.65
Transfer/sec:      0.94MB
```

---

## Comparativa de Resultados

| Framework    | Requests/sec | Latencia Promedio |
|--------------|--------------|-------------------|
| Bunnyhopapi  | **11358.95** | 17.83ms          |
| FastAPI      | 2059.83      | 198.46ms         |
| Django       | 3398.65      | 116.12ms         |

**Bunnyhopapi es 5.46 veces más rápido que FastAPI y 3.34 veces más rápido que Django.**

---


