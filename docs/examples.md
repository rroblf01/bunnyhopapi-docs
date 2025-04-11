## Add Authentication to the API

```python
from bunnyhopapi.server import Server, Router
from bunnyhopapi.models import Endpoint
import logging

logger = logging.getLogger(__name__)


class AuthEndpoint(Endpoint):
    path = "/auth"

    @Endpoint.GET()
    def get(self, headers):
        return 200, {"message": "GET /auth"}


def auth_middleware(headers, endpoint, *args, **kwargs):
    logger.info("auth_middleware: Before calling the endpoint")
    if "Authorization" not in headers:
        return 401, {"message": "Unauthorized"}
    logger.info("auth_middleware: After calling the endpoint")
    return endpoint(headers=headers, *args, **kwargs)


if __name__ == "__main__":
    server = Server()

    auth_router = Router(middleware=auth_middleware)
    auth_router.include_endpoint_class(AuthEndpoint)
    server.include_router(auth_router)
    server.run()
```

## CRUD with sqlite3

```python
from pydantic import BaseModel
from bunnyhopapi.server import Server, Router
from bunnyhopapi.models import Endpoint, PathParam
import sqlite3
from contextlib import contextmanager
import uuid
import logging

logger = logging.getLogger(__name__)


class UserInput(BaseModel):
    name: str
    age: int


class UserOutput(BaseModel):
    id: str
    name: str
    age: int


class Message(BaseModel):
    message: str


class UserList(BaseModel):
    users: list[UserOutput]


class Database:
    def __init__(self, db_name="users.db"):
        self.db_name = db_name
        self.create_table()

    def create_table(self):
        with self.get_connection() as conn:
            conn.execute("""
                CREATE TABLE IF NOT EXISTS users (
                    id TEXT PRIMARY KEY,
                    name TEXT NOT NULL,
                    age INTEGER NOT NULL
                )
            """)

    @contextmanager
    def get_connection(self):
        conn = sqlite3.connect(self.db_name)
        try:
            yield conn
        finally:
            conn.close()

    def add_user(self, user: UserInput) -> UserOutput:
        user_id = str(uuid.uuid4())
        new_user = UserOutput(id=user_id, **user.model_dump())
        with self.get_connection() as conn:
            conn.execute(
                """
                INSERT INTO users (id, name, age)
                VALUES (?, ?, ?)
            """,
                (new_user.id, new_user.name, new_user.age),
            )
            conn.commit()
        return new_user

    def get_users(self) -> list[UserOutput]:
        with self.get_connection() as conn:
            cursor = conn.execute("SELECT id, name, age FROM users")
            users = [UserOutput(id=row[0], name=row[1], age=row[2]) for row in cursor]
        return users

    def get_user(self, user_id: str) -> UserOutput | None:
        with self.get_connection() as conn:
            cursor = conn.execute(
                "SELECT id, name, age FROM users WHERE id = ?", (user_id,)
            )
            row = cursor.fetchone()
        if row:
            return UserOutput(id=row[0], name=row[1], age=row[2])
        return None

    def update_user(self, user_id: str, user: UserInput) -> UserOutput | None:
        with self.get_connection() as conn:
            cursor = conn.execute(
                """
                UPDATE users
                SET name = ?, age = ?
                WHERE id = ?
            """,
                (user.name, user.age, user_id),
            )
            conn.commit()
            if cursor.rowcount == 0:
                return None
            return self.get_user(user_id)

    def delete_user(self, user_id: str) -> bool:
        with self.get_connection() as conn:
            cursor = conn.execute("DELETE FROM users WHERE id = ?", (user_id,))
            conn.commit()
            return cursor.rowcount > 0


class UserEndpoint(Endpoint):
    path: str = "/users"

    @Endpoint.MIDDLEWARE()
    def db_middleware(self, endpoint, headers, *args, **kwargs):
        logger.info("db_middleware: Before calling the endpoint")
        db = Database()
        return endpoint(headers=headers, db=db, *args, **kwargs)

    @Endpoint.GET()
    def get(self, headers, db: Database, *args, **kwargs) -> {200: UserList}:
        users = db.get_users()
        return 200, {"users": users}

    @Endpoint.GET()
    def get_with_params(
        self, db, user_id: PathParam[str], headers, *args, **kwargs
    ) -> {200: UserOutput, 404: Message}:
        users = db.get_user(user_id)

        if users is None:
            return 404, {"message": "User not found"}

        return 200, users

    @Endpoint.POST()
    def post(self, user: UserInput, headers, db, *args, **kwargs) -> {201: UserOutput}:
        new_user = db.add_user(user)
        return 201, new_user

    @Endpoint.PUT()
    def put(
        self, db, user_id: PathParam[str], user: UserInput, headers, *args, **kwargs
    ) -> {200: UserOutput, 404: Message}:
        updated_user = db.update_user(user_id, user)

        if updated_user is None:
            return 404, {"message": "User not found"}

        return 200, updated_user

    @Endpoint.DELETE()
    def delete(
        self, db, user_id: PathParam[str], headers, *args, **kwargs
    ) -> {200: Message, 404: Message}:
        if db.delete_user(user_id):
            return 200, {"message": "User deleted"}
        else:
            return 404, {"message": "User not found"}


if __name__ == "__main__":
    server = Server()
    auth_router = Router()
    auth_router.include_endpoint_class(UserEndpoint)
    server.include_router(auth_router)
    server.run(workers=1)
```

## SSE
Create the HTML file `sse_index.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <ul id="messages">
        
    </ul>

    <script>
        const host = window.location.hostname || "localhost";
        const eventSource = new EventSource(`http://${host}:8000/sse/events`);
    
    eventSource.addEventListener('start', function(event) {
        addMessage(`Start: ${event.data}`);
    });
    
    eventSource.addEventListener('progress', function(event) {
        addMessage(`Progress: ${event.data}`);
    });
    
    eventSource.addEventListener('complete', function(event) {
        addMessage(`Complete: ${event.data}`);
    });
    
    eventSource.addEventListener('end', function(event) {
        addMessage(`End: ${event.data}`);
        eventSource.close();
    });
    
    eventSource.onerror = function(event) {
        console.error("EventSource failed:", event);
        addMessage("Error occurred");
        eventSource.close();
    };
    
    function addMessage(message) {
        const li = document.createElement('li');
        li.textContent = message;
        document.getElementById('messages').appendChild(li);
    }
    </script>
</body>
</html>
```

and the following `main.py`
```python
from bunnyhopapi.server import Server
from bunnyhopapi.models import Endpoint
import asyncio
from bunnyhopapi.templates import (
    serve_static_file,
)


class SseEndpoint(Endpoint):
    path = "/sse/events"

    @Endpoint.GET(content_type=Server.CONTENT_TYPE_SSE)
    async def get(self, headers) -> {200: str}:
        events = ["start", "progress", "complete"]
        for event in events:
            yield f"event: {event}\ndata: Processing {event}\n\n"
            await asyncio.sleep(1.5)
        yield "event: end\ndata: Processing complete\n\n"


class SseTemplateEndpoint(Endpoint):
    path = "/sse"

    @Endpoint.GET(content_type=Server.CONTENT_TYPE_HTML)
    async def get(self, headers):
        return await serve_static_file("sse_index.html")


def main():
    server = Server(cors=True)

    server.include_endpoint_class(SseEndpoint)
    server.include_endpoint_class(SseTemplateEndpoint)

    server.run()


if __name__ == "__main__":
    main()
```

## WebSocket
Create the following HTML file `ws_index.html`
```html
<!DOCTYPE html>
<html>
<head>
    <title>WebSocket Client</title>
</head>
<body>
    <h1>WebSocket Test</h1>
    <button onclick="connect()">Connect</button>
    <button onclick="disconnect()">Disconnect</button>
    <input type="text" id="messageInput">
    <button onclick="sendMessage()">Send</button>
    <div id="output"></div>

    <script>
        let socket;
        const host = window.location.hostname || 'localhost';
        const wsUrl = `ws://${host}:8000/ws/chat`;
        
        function connect() {
            socket = new WebSocket(wsUrl);
            
            socket.onopen = function(e) {
                log("Connected to WebSocket server");
            };
            
            socket.onmessage = function(event) {
                log("Received: " + event.data);
            };
            
            socket.onclose = function(event) {
                log("Connection closed");
            };
            
            socket.onerror = function(error) {
                log("Error: " + error.message);
            };
        }
        
        function disconnect() {
            if (socket) {
                socket.close();
            }
        }
        
        function sendMessage() {
            const input = document.getElementById("messageInput");
            const message = input.value;
            
            if (!socket || socket.readyState !== WebSocket.OPEN) {
                log("Not connected");
                return;
            }
            
            socket.send(message);
            input.value = "";
        }
        
        function log(message) {
            const output = document.getElementById("output");
            output.innerHTML += `<p>${message}</p>`;
        }
    </script>
</body>
</html>
```

and the following `main.py`
```python
from bunnyhopapi.server import Server
from bunnyhopapi import logger
from bunnyhopapi.models import Endpoint
import asyncio
from bunnyhopapi.templates import (
    serve_static_file,
)


class WSEndpoint(Endpoint):
    path = "/ws/chat"

    @Endpoint.MIDDLEWARE()
    async def class_middleware(self, endpoint, headers, **kwargs):
        logger.info("middleware: Before calling the endpoint")
        async for response in endpoint(headers=headers, **kwargs):
            yield response
        logger.info("middleware: After calling the endpoint")

    async def connection(self, headers):
        logger.info("Client connected")
        return True

    async def disconnect(self, connection_id, headers):
        logger.info(f"Client {connection_id} disconnected")

    async def ws(self, connection_id, message, headers):
        logger.info(f"Received message from {connection_id}: {message}")
        for i in range(10):
            yield f"event: message\ndata: {i}\n\n"
            await asyncio.sleep(0.2)


class WSTemplateEndpoint(Endpoint):
    path = "/ws"

    @Endpoint.GET(content_type=Server.CONTENT_TYPE_HTML)
    async def get(self, headers):
        return await serve_static_file("ws_index.html")


def main():
    server = Server(cors=True)

    server.include_endpoint_class(WSEndpoint)
    server.include_endpoint_class(WSTemplateEndpoint)
    server.run()


if __name__ == "__main__":
    main()
```

## Jinja2
Create the following files in `templates/`:
- `base.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Default Title{% endblock %}</title>
    <link rel="stylesheet" href="/static/css/styles.css">
</head>
<body>
    <header>
        <h1>{% block header %}Welcome to My Website{% endblock %}</h1>
    </header>
    <nav>
        {% block nav %}
        <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/about">About</a></li>
            <li><a href="/contact">Contact</a></li>
        </ul>
        {% endblock %}
    </nav>
    <main>
        {% block content %}
        <p>This is the default content.</p>
        {% endblock %}
    </main>
    <footer>
        {% block footer %}
        <p>&copy; 2023 My Website</p>
        {% endblock %}
    </footer>
</body>
</html>
```
- `components.html`
```html
{% macro button(label, url, css_class="btn") %}
<a href="{{ url }}" class="{{ css_class }}">{{ label }}</a>
{% endmacro %}

{% macro alert(message, alert_type="info") %}
<div class="alert alert-{{ alert_type }}">
    {{ message }}
</div>
{% endmacro %}

{% macro card(title, content, footer=None) %}
<div class="card">
    <div class="card-header">
        <h3>{{ title }}</h3>
    </div>
    <div class="card-body">
        <p>{{ content }}</p>
    </div>
    {% if footer %}
    <div class="card-footer">
        {{ footer }}
    </div>
    {% endif %}
</div>
{% endmacro %}
```

- `index.html`
```html
{% extends "base.html" %}
{% import "components.html" as components %}

{% block title %}Test Page{% endblock %}

{% block header %}Testing Jinja2 Components{% endblock %}

{% block content %}
    <h2>Buttons</h2>
    {{ components.button("Home", "/", "btn btn-primary") }}
    {{ components.button("About", "/about", "btn btn-secondary") }}

    <h2>Alerts</h2>
    {{ components.alert("This is an info alert.") }}
    {{ components.alert("This is a warning alert.", "warning") }}

    <h2>Cards</h2>
    {{ components.card("Card Title", "This is the card content.", "Footer text") }}
    {{ components.card("Another Card", "Content without footer.") }}
{% endblock %}
```

and the following `main.py`
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
        self.template_env = create_template_env("templates/")

    @Endpoint.GET(content_type=Server.CONTENT_TYPE_HTML)
    async def get(self, headers):
        return await render_jinja_template("index.html", self.template_env)


def main():
    server = Server(cors=True)

    static_folder = os.path.join(os.path.dirname(__file__), "static")
    server.include_static_folder(static_folder)

    server.include_endpoint_class(JinjaTemplateEndpoint)
    server.run()


if __name__ == "__main__":
    main()
```