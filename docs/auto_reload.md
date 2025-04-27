# Server Auto-Reload

BunnyHopAPI allows enabling server auto-reload by using the parameter `auto_reload=True` when initializing the server. This is especially useful during development, as every time you save a `.py` file, the server will automatically reload, reflecting the changes made.

## Example

```python
from bunnyhopapi.server import Server

if __name__ == "__main__":
    server = Server(auto_reload=True)
    server.run()
```

## Explanation

In this example, we have configured the server with `auto_reload=True`. This means that:

- Every time a change is detected in a `.py` file within the project, the server will automatically restart.
- It is not necessary to manually stop and restart the server to apply the changes.

This functionality significantly improves productivity during development, allowing for quick and efficient iterations.

> **Note:** Auto-reload is designed for development environments and should not be used in production.
