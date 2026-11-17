# VYS-Request

Colección de peticiones HTTP (archivos `.rest`) para probar la API de **Vida y Salud** (`vidaysalud.com`) con la extensión [REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) de VS Code.

## Estructura

```
api/
├── .env                      # Credenciales Production + Stage (NO se versiona)
├── Production/
│   ├── api/content.rest      # API con login: shows, episodios, watch
│   └── web/                  # content, diagnosis, geo, health
└── Stage/
    └── web/                  # content, diagnosis, geo, health
```

## Configuración

1. Instalar la extensión **REST Client** (`humao.rest-client`).
2. Crear un único `api/.env` con las variables de ambos entornos (prefijo `VYS_PROD_` / `VYS_STAGE_`):

```env
VYS_PROD_EMAIL=
VYS_PROD_PASSWORD=
VYS_PROD_WEB_API_KEY=
VYS_PROD_GEO_API_KEY=
VYS_PROD_HEALTH_SECRET=

VYS_STAGE_EMAIL=
VYS_STAGE_PASSWORD=
VYS_STAGE_WEB_API_KEY=
VYS_STAGE_GEO_API_KEY=
VYS_STAGE_HEALTH_SECRET=
```

| Variable (`VYS_PROD_*` / `VYS_STAGE_*`) | Uso                                                  |
| ------------------- | ---------------------------------------------------- |
| `EMAIL`         | Email para `POST /api/login`                         |
| `PASSWORD`      | Contraseña para `POST /api/login`                    |
| `WEB_API_KEY`   | API key web (`x-api-key` o `Authorization: Bearer`)  |
| `GEO_API_KEY`   | API key de `/geo`                                    |
| `HEALTH_SECRET` | Header `x-health-secret` de `/api/health`            |

> ⚠️ No subir `.env` al repositorio; agregarlo a `.gitignore`.

## Uso

Abrir un `.rest` y pulsar **Send Request** sobre la petición deseada (se separan con `###`). Las variables `{{$dotenv NOMBRE}}` se leen del `.env`.

## Endpoints

Base: `https://api.vidaysalud.com`

### `api/content.rest` (con login)

Se hace login primero y el token se reutiliza (`{{login.response.body.data.token}}`).

| Método | Ruta                 | Descripción           |
| ------ | -------------------- | --------------------- |
| POST   | `/api/login`         | Obtiene el token      |
| GET    | `/api/shows`         | Lista de shows        |
| GET    | `/api/episodes/{id}` | Episodios de un show  |
| GET    | `/api/watch/{id}`    | Datos de reproducción |

### `web/*.rest`

| Método | Ruta                           | Auth                    | Descripción                      |
| ------ | ------------------------------ | ----------------------- | -------------------------------- |
| POST   | `/api/web/shows`               | `x-api-key`             | Shows                            |
| POST   | `/api/web/episodes/{id}`       | `x-api-key`             | Episodios de un show             |
| POST   | `/api/web/episodes/{id}/video` | `x-api-key`             | Video de un episodio             |
| POST   | `/geo`                         | `Authorization: Bearer` | Geolocalización                  |
| GET    | `/api/health`                  | `x-health-secret`       | Estado del servicio              |
| POST   | `/api/send-diagnosis`          | `Authorization: Bearer` | Guarda diagnóstico y envía correo |

> ℹ️ `health.rest` y la primera petición de `geo.rest` apuntan hoy a `https://api-stage.vidaysalud.com`, no a producción.

### `POST /api/send-diagnosis`

Body: `jsonDiag` (objeto, obligatorio) y `pin` (opcional).

| Caso                    | Resultado esperado                                 |
| ----------------------- | -------------------------------------------------- |
| Con `pin`               | `200` — guarda en Google Sheets y **envía correo** |
| Sin `pin`               | `200` — guarda la fila, no envía correo            |
| Falta `jsonDiag`        | `422`                                              |
| `jsonDiag` no es objeto | `422`                                              |
| Sin API key             | `403` "Authentication error"                       |
| API key inválida        | `401` "Invalid API key"                            |

> ⚠️ Estamos en **producción**: la petición con `pin` envía un correo real a los destinatarios de `senddiagnosis.emails` y ambas peticiones válidas escriben filas reales en Google Sheets.
