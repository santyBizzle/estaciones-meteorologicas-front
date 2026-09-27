# estaciones-meteorologicas-front

## Configuración 'Rewrites and Redirects' en AWS Amplify

> **Disclaimer:** Esta sección fue escrita describiendo los pasos de acuerdo a la fecha en la que fue redactada, pueden existir diferencias de acuerdo a la fecha en la que esté leyendo esto.

Pasos:

1. En AWS Amplify luego de configurar una app con el repositorio del frontend.
2. Seleccionar la app ir a **Hosting** --> **Environment variables** y agregar la variable `VUE_APP_AXIOS_URL` con el valor de `/api`
3. En **Hosting** --> **Rewrites and redirects** colocar los siguientes redirects:
```json
[
  {
    "source": "/api/<*>",
    "status": "200",
    "target": "https://<DNS-DEL-ALB>/api/<*>"
  },
  {
    "source": "/<*>",
    "status": "404-200",
    "target": "/index.html"
  }
]
```

**IMPORTANTE:** En `<DNS-DEL-ALB>` reemplace con el **DNS del balanceador de carga del backend** ya que no se puede colocar variables en esta sección.

**OJO** el target tiene que ser **HTTPS** por lo que necesita tener un certificado SSL, una opción es tener un dominio propio por ejemplo en `Cloudflare` y hacer un `proxy` hacia el DNS del load balancer en AWS.

Ejemplo:

```json
[
  {
    "source": "/api/<*>",
    "status": "200",
    "target": "https://puyu-api.numero17.dev/api/<*>"
  },
  {
    "source": "/<*>",
    "status": "404-200",
    "target": "/index.html"
  }
]
```
