# Plantillas y Archivos Estáticos en BunnyHopAPI

BunnyHopAPI permite servir archivos estáticos, renderizar archivos HTML estáticos y usar Jinja2 para generar contenido dinámico. A continuación, se explican estas funcionalidades con ejemplos.

---

## Servir Archivos Estáticos

Puedes servir archivos estáticos como imágenes, CSS o JavaScript utilizando el método `include_static_folder`. Esto es útil para aplicaciones que necesitan recursos estáticos.

### Ejemplo

```python
from bunnyhopapi.server import Server
import os

def main():
    server = Server(cors=True)

    # Ruta a la carpeta de archivos estáticos
    static_folder = os.path.join(os.path.dirname(__file__), "static")
    server.include_static_folder(static_folder)

    server.run()

if __name__ == "__main__":
    main()
```

En este ejemplo, todos los archivos dentro de la carpeta `static` estarán disponibles para ser servidos.

---

## Renderizar un HTML Estático

Puedes renderizar un archivo HTML estático utilizando el método `serve_static_file`. Esto es útil para páginas que no requieren contenido dinámico.

### Ejemplo

```python
from bunnyhopapi.server import Server
from bunnyhopapi.models import Endpoint
from bunnyhopapi.templates import serve_static_file

class StaticHtmlEndpoint(Endpoint):
    path = "/static-page"

    @Endpoint.GET(content_type=Server.CONTENT_TYPE_HTML)
    async def get(self, headers):
        return await serve_static_file("example/templates/static_html/static_page.html")

def main():
    server = Server(cors=True)
    server.include_endpoint_class(StaticHtmlEndpoint)
    server.run()

if __name__ == "__main__":
    main()
```

En este ejemplo, se sirve un archivo HTML estático ubicado en `example/templates/static_html/static_page.html`.

---

## Usar Jinja2 para Plantillas Dinámicas

Jinja2 permite generar contenido dinámico en tus páginas HTML. BunnyHopAPI facilita la integración de Jinja2 mediante los métodos `create_template_env` y `render_jinja_template`.

### Ejemplo

```python
from bunnyhopapi.server import Server
from bunnyhopapi.models import Endpoint
import os
from bunnyhopapi.templates import (
    render_jinja_template,
    create_template_env,
)

class JinjaTemplateEndpoint(Endpoint):
    path = "/"

    def __init__(self):
        super().__init__()
        self.template_env = create_template_env("example/templates/jinja/")

    @Endpoint.GET(content_type=Server.CONTENT_TYPE_HTML)
    async def get(self, headers):
        return await render_jinja_template("index.html", self.template_env)

def main():
    server = Server(cors=True)

    # Ruta a la carpeta de archivos estáticos
    static_folder = os.path.join(os.path.dirname(__file__), "static")
    server.include_static_folder(static_folder)

    server.include_endpoint_class(JinjaTemplateEndpoint)
    server.run()

if __name__ == "__main__":
    main()
```

En este ejemplo:
- Se configura un entorno de plantillas Jinja2 en la carpeta `example/templates/jinja/`.
- Se renderiza dinámicamente el archivo `index.html` utilizando Jinja2.
- También se incluye una carpeta de archivos estáticos.

---

Con estas funcionalidades, BunnyHopAPI te permite manejar tanto contenido estático como dinámico de manera eficiente.
