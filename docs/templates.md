# Templates and Static Files in BunnyHopAPI

BunnyHopAPI allows serving static files, rendering static HTML files, and using Jinja2 to generate dynamic content. Below, these functionalities are explained with examples.

---

## Serving Static Files

You can serve static files such as images, CSS, or JavaScript using the `include_static_folder` method. This is useful for applications that need static resources.

### Example

```python
from bunnyhopapi.server import Server
import os

def main():
    server = Server(cors=True)

    # Path to the static files folder
    static_folder = os.path.join(os.path.dirname(__file__), "static")
    server.include_static_folder(static_folder)

    server.run()

if __name__ == "__main__":
    main()
```

In this example, all files inside the `static` folder will be available to be served.

---

## Rendering a Static HTML

You can render a static HTML file using the `serve_static_file` method. This is useful for pages that do not require dynamic content.

### Example

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

In this example, a static HTML file located at `example/templates/static_html/static_page.html` is served.

---

## Using Jinja2 for Dynamic Templates

Jinja2 allows generating dynamic content in your HTML pages. BunnyHopAPI facilitates the integration of Jinja2 through the `create_template_env` and `render_jinja_template` methods.

### Example

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

    # Path to the static files folder
    static_folder = os.path.join(os.path.dirname(__file__), "static")
    server.include_static_folder(static_folder)

    server.include_endpoint_class(JinjaTemplateEndpoint)
    server.run()

if __name__ == "__main__":
    main()
```

In this example:
- A Jinja2 template environment is configured in the folder `example/templates/jinja/`.
- The file `index.html` is dynamically rendered using Jinja2.
- A static files folder is also included.

---

With these functionalities, BunnyHopAPI allows you to efficiently handle both static and dynamic content.
