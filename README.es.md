# zerollama-dashboard

Una página de estado en vivo para tu servidor
[llama.cpp](https://github.com/ggml-org/llama.cpp). Ábrela en el
navegador, observa lo que hace tu modelo y chatea con él desde la
misma página.

[**▶ Probar la demo en vivo**](https://jungrok5.github.io/zerollama-dashboard/?demo=1) — funciona sin un servidor real, solo para probarla.

> Languages: [English](README.md) · [한국어](README.ko.md) · [日本語](README.ja.md) · [简体中文](README.zh-CN.md) · **Español**

![panel](docs/screenshots/main-en.png)
![panel de chat](docs/screenshots/chat-panel.png)
![vista móvil](docs/screenshots/mobile.png)

## Qué obtienes

- **Vista en tiempo real** — velocidad de generación, cola de
  peticiones y actividad de slots actualizándose en directo.
- **Chatea con tu modelo** dentro del propio panel. Respuesta en
  streaming, render de Markdown, botón de detener, y sliders para
  temperature / top_p / max_tokens.
- **Consejos en lenguaje claro** cuando el servidor sufre — por
  ejemplo, "la cola está creciendo, prueba `--parallel +1`".
- **Detalle por slot**, incluyendo toda la configuración de muestreo
  (temperature, top_k, top_p, repeat_penalty, mirostat, …).
- **Cinco idiomas de interfaz**: English / 한국어 / 日本語 /
  简体中文 / Español. Se elige uno desde el navegador y puedes
  cambiarlo desde la cabecera en cualquier momento.
- **Servidores single-model y router** ambos detectados
  automáticamente.

## Puesta en marcha

Necesitas un llama-server razonablemente reciente, arrancado con
`--metrics`. Lo más simple es dejar que el propio llama-server sirva
`monitor.html`:

```bash
mkdir -p public
cp monitor.html public/
llama-server -m model.gguf --metrics --port 8080 --path ./public
```

Luego abre `http://localhost:8080/monitor.html`.

Para monitorizar un servidor en otra máquina, sirve `monitor.html`
desde cualquier hosting estático y pasa `?server=`:

```
http://localhost:8000/monitor.html?server=http://10.0.0.5:8080
```

## Opciones de URL

| Parámetro | Qué hace |
|---|---|
| `server` | URL de llama-server (por defecto: mismo origen) |
| `model` | modo router: modelo preseleccionado |
| `lang` | idioma de la UI (`en` / `ko` / `ja` / `zh-CN` / `es`) |
| `prompt` | rellena el campo de chat al cargar |
| `demo` | `1` o `router` para el servidor mock integrado |
| `poll` | intervalo de polling, ms (por defecto `1000`) |
| `log` | ruta del archivo de log (autodetectada si se omite) |

Los ajustes viven solo en la URL — comparte un enlace y quien lo abra
verá lo mismo.

## Aloja tu propia demo

¿Quieres una URL pública que cualquiera pueda abrir? GitHub Pages es
gratis y no se duerme:

1. **Settings → Pages → Source = "GitHub Actions"**.
2. Push a `main`.

El workflow incluido (`.github/workflows/pages.yml`) construye y
despliega automáticamente. Los visitantes acceden a
`https://<tu-usuario>.github.io/<repo>/?demo=1`. Todo es estático,
no hay backend que mantener.

## Privacidad y seguridad

- El panel nunca envía datos a otra parte que no sea la URL de
  llama-server que tú indiques (y, opcionalmente, una ruta de log en
  el mismo origen).
- No usa cookies, localStorage ni ningún tipo de tracking.
- Cualquier acción que cambie el estado del servidor — cargar o
  descargar un modelo, guardar / restaurar / borrar el KV de un slot
  — pide confirmación primero.
- El render de Markdown es seguro: nada que escriba el modelo puede
  escapar como HTML o script.

## Licencia

[MIT](LICENSE). Inspirado en
[abhiFSD/llama.cpp-Monitor-Dashboard](https://github.com/abhiFSD/llama.cpp-Monitor-Dashboard).
¿Vas a contribuir? Lee [CLAUDE.md](CLAUDE.md).
