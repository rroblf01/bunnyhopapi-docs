# Instalación y Uso de BunnyHopApi

---

## Instalación


**Instalación desde PyPI**:

   Puedes instalar BunnyHopApi directamente desde el índice de paquetes de Python (PyPI) utilizando `pip`. Abre tu terminal y ejecuta el siguiente comando:

   ```bash
   pip install bunnyhopapi
   ```


---

## Ejemplo de Uso

A continuación, se muestra un ejemplo básico de cómo configurar y usar BunnyHopApi para crear un endpoint HTTP simple.

### Ejemplo: Endpoint HTTP

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

## Explicación del Código

1. **Importación de Módulos**:
   - Importamos `Server` para crear el servidor.
   - Importamos `Endpoint` para definir nuestros endpoints.

2. **Definición del Endpoint**:
   - Creamos una clase `HealthEndpoint` que hereda de `Endpoint`.
   - Definimos la ruta del endpoint con `path = "/health"`.
   - Implementamos el un método con el decorador `GET` y hacemos que devuelva un código de estado y un JSON.

3. **Configuración del Server**:
   - Creamos una instancia de `Server`.
   - Añadimos el `HealthEndpoint` al servidor.

4. **Ejecución del Servidor**:
   - Ejecutamos el servidor llamando a `server.run()`.

---

Con estos pasos, tendrás un servidor básico en funcionamiento que responde a las solicitudes HTTP en el endpoint `/health`.
Por defecto se ejecutará con el host `localhost` y el puerto `8000`. Puedes acceder a él abriendo tu navegador y navegando a `http://localhost:8000/health`.
También puedes usar la documentación Swagger generada automáticamente para explorar y probar tu API en `http://localhost:8000/docs`.
¡Feliz desarrollo con BunnyHopApi!

<script>
  (function() {
    const linkElm = document.querySelector('#template a[download="index.html"]');
    const codeElm = document.querySelector('#template code');
    const html = codeElm?.textContent;

    linkElm?.setAttribute('href', `data:text/plain,${html}`);
  })();
</script>