# Recarga Automática del Servidor

BunnyHopAPI permite habilitar la recarga automática del servidor utilizando el parámetro `auto_reload=True` al inicializar el servidor. Esto es especialmente útil durante el desarrollo, ya que cada vez que guardes un archivo `.py`, el servidor se recargará automáticamente, reflejando los cambios realizados.

## Ejemplo

```python
from bunnyhopapi.server import Server

if __name__ == "__main__":
    server = Server(auto_reload=True)
    server.run()
```

## Explicación

En este ejemplo, hemos configurado el servidor con `auto_reload=True`. Esto significa que:

- Cada vez que se detecte un cambio en un archivo `.py` dentro del proyecto, el servidor se reiniciará automáticamente.
- No es necesario detener y volver a iniciar manualmente el servidor para aplicar los cambios.

Esta funcionalidad mejora significativamente la productividad durante el desarrollo, permitiendo iteraciones rápidas y eficientes.

> **Nota:** La recarga automática está diseñada para entornos de desarrollo y no debe usarse en producción.
