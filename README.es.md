# zerollama-dashboard

Un panel de monitoreo de un único archivo HTML, sin dependencias, para
servidores [llama.cpp](https://github.com/ggml-org/llama.cpp).

> Languages: [English](README.md) · [한국어](README.ko.md) · [日本語](README.ja.md) · [简体中文](README.zh-CN.md) · **Español**

> Inspirado en [abhiFSD/llama.cpp-Monitor-Dashboard](https://github.com/abhiFSD/llama.cpp-Monitor-Dashboard) (MIT).
> Sin npm, sin CDN, sin localStorage. Solo un archivo HTML.

## Capturas

![panel (inglés)](docs/screenshots/main-en.png)
![panel (coreano)](docs/screenshots/main-ko.png)
![detalle de slots](docs/screenshots/slots-open.png)
![vista móvil](docs/screenshots/mobile.png)

## Qué muestra

- `/metrics` en vivo (Prometheus): tok/s de generación / prompt,
  peticiones en proceso / encoladas, slots ocupados por decode,
  contadores acumulados de prompt y generación
- `/slots`: estado por slot con todos los parámetros de muestreo
  (temperature, top_k, top_p, min_p, repeat_penalty, mirostat, DRY, etc.)
- `/props` + `/v1/models`: metadatos del modelo (vocab/contexto/embedding,
  parámetros, plantilla de chat, modalidades, info de build)
- `/lora-adapters`: adaptadores LoRA cargados con escalas
- `/models` (modo router): todos los modelos en caché con estado (loaded /
  loading / unloaded / sleeping / failed) y **los argumentos CLI reales
  con los que se lanzó cada modelo**
- `server.log` opcional vía HTTP Range (autodetectado; oculto si no
  está disponible)
- **Guía en línea**: cada tarjeta lleva un tooltip ⓘ que explica el
  parámetro subyacente; aparecen sugerencias cuando el estado cruza
  umbrales
- **Panel de chat integrado**: streaming de `POST /v1/chat/completions`,
  prompt de sistema, sliders de parámetros, botón de cancelar y modo
  fan-out paralelo para pruebas de saturación de slots (más abajo)

## Inicio rápido

### Opción A — mismo origen que llama-server

```bash
mkdir -p ./public
cp monitor.html ./public/
llama-server -m model.gguf --metrics --port 8080 --path ./public
# abre http://localhost:8080/monitor.html
```

### Opción B — apuntar a un servidor remoto

Sirve `monitor.html` desde cualquier servidor estático (p.ej.
`python3 -m http.server`) y pasa `?server=`:

```
http://localhost:8000/monitor.html?server=http://10.0.0.5:8080
```

El `llama-server` remoto debe permitir CORS (por defecto es permisivo).

### Modo router (multi-modelo)

Lanza `llama-server` **sin** `-m`:

```bash
llama-server --models-dir ./models --metrics --port 8080
```

El panel detecta automáticamente el modo router probando `GET /models`.
Aparece un selector de modelo en la cabecera.

## Parámetros de URL

| Param | Por defecto | Uso |
|---|---|---|
| `server` | mismo origen | URL base de llama-server |
| `model` | (ninguno) | modo router: modelo seleccionado por defecto |
| `poll` | `1000` | intervalo de polling (ms) |
| `log` | auto | ruta del log; autodetectado si no se especifica, panel oculto si no es accesible |
| `lang` | auto | `en` / `ko` / `ja` / `zh-CN` / `es` (por defecto desde el navegador) |
| `prompt` | (ninguno) | rellena el campo del chat al cargar (no envía automáticamente) |

Los ajustes viven solo en la URL — sin localStorage. Comparte un enlace,
obtén la misma vista.

## Panel de chat

Entre la grilla de slots y la tarjeta del modelo hay un panel de chat
plegable. Habla con el mismo llama-server que estás monitorizando, así
que puedes lanzar un prompt y ver cómo reaccionan las métricas en tiempo
real.

- **Streaming**: SSE desde `/v1/chat/completions`; los tokens aparecen
  según llegan. El botón de detener aborta el stream activo.
- **Renderizado Markdown**: encabezados, párrafos, **negrita**,
  *cursiva*, `code` en línea, bloques de código, listas ordenadas /
  no ordenadas, tablas (GFM), citas, líneas horizontales, y enlaces
  `[texto](https://…)`. Sin librería externa: el renderizador construye
  nodos DOM directamente con `textContent`, así la salida del servidor
  no puede inyectar HTML ni scripts.
- **Prompt de sistema**: opcional. Se envía como `role: "system"` antes
  del historial.
- **Sliders**: `temperature`, `top_p`, `max_tokens` y un fan-out
  paralelo (1–8) que dispara el mismo prompt a N slots simultáneamente
  para una prueba rápida de saturación.
- **Sin persistencia**: por regla del proyecto no hay localStorage, así
  que recargar la página borra la conversación. Usa `?prompt=…` para
  prerellenar la entrada desde la URL.

## Opcional: tail de logs

El panel de logs lee los últimos ~64 KB de un archivo estático mediante
`Range: bytes=-65536`. Para que funcione, **las tres** condiciones deben
cumplirse:

1. La salida de llama-server se redirige a un archivo:
   ```bash
   llama-server ... --path ./public > ./public/server.log 2>&1
   ```
   (systemd / docker por defecto solo van a stdout — no se crea archivo.)
2. Ese archivo se sirve desde el mismo origen que `monitor.html`.
3. El servidor HTTP soporta `Range` (cpp-httplib y nginx lo hacen).

Si alguna condición falla, el panel se oculta silenciosamente. Anula la
ruta con `?log=path/to/file`. Desactívalo explícitamente con `?log=`.

## Requisitos en llama-server

- Build / binario lo bastante reciente para exponer `/metrics`, `/slots`,
  `/props`, `/v1/models`, `/lora-adapters` (todos estándar).
- Ejecutar con `--metrics` para exponer `/metrics`.
- `--slots` está activo por defecto; no pases `--no-slots`.
- Para modo router: lanza sin `-m`, con `--models-dir` o
  `--models-preset`.

## Reglas de guía (extracto)

| Señal | Umbral | Sugerencia |
|---|---|---|
| Slots ocupados / decode | ≥ 90% de total_slots sostenido | Sube `--parallel` o reduce la concurrencia del cliente |
| Peticiones encoladas | > 0 sostenido | Sube `--parallel` o baja la concurrencia del cliente |
| tok/s bajo + slots inactivos | — | Aumenta `--n-gpu-layers` |
| `is_sleeping` true | — | La próxima petición recargará el modelo — ajusta `--sleep-idle-seconds` |
| Slot `temperature` 0 | — | Decodificación greedy (determinista) |
| Slot `temperature` > 1.5 | — | La calidad puede caer; ≤1.0 típico |
| Slot `repeat_penalty` > 1.3 | — | Puede romper formatos; 1.05–1.15 típico |
| Slot `mirostat` ≠ 0 | — | top_p / top_k se ignoran |
| Modelo router `failed` | — | Inspecciona `exit_code`; revisa args / VRAM |

El conjunto completo de reglas se renderiza en el panel "Active
suggestions" del dashboard cuando se activan.

## Licencia

[MIT](LICENSE). Inspirado en
[abhiFSD/llama.cpp-Monitor-Dashboard](https://github.com/abhiFSD/llama.cpp-Monitor-Dashboard).
