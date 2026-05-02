# zerollama-dashboard

Un panel de monitoreo en un único archivo HTML para servidores
[llama.cpp](https://github.com/ggml-org/llama.cpp). Muestra en
tiempo real el estado de cada slot y sus parámetros de muestreo, y
te dice en lenguaje claro qué opción ajustar cuando algo necesita
afinarse. Un chat integrado en la misma página te permite lanzar
prompts y observar cómo reaccionan las métricas.

[**▶ Probar la demo en vivo**](https://jungrok5.github.io/zerollama-dashboard/?demo=1) — corre contra un servidor mock integrado, sin backend.

> Languages: [English](README.md) · [한국어](README.ko.md) · [日本語](README.ja.md) · [简体中文](README.zh-CN.md) · **Español**

![panel](docs/screenshots/main-en.png)
![panel de chat](docs/screenshots/chat-panel.png)

## Por qué existe

La WebUI integrada de llama.cpp es un cliente de chat y no muestra el
estado del servidor en vivo. Las pilas Prometheus + Grafana exponen
números pero no te dicen qué flag tocar. Open WebUI, LibreChat y
similares son frontends de chat, no monitores conscientes de los
slots. zerollama-dashboard se sitúa en ese hueco: una vista de operador
de `/metrics` + `/slots` con consejos de ajuste prescriptivos,
soporte de modo **router** multi-modelo (incluyendo los argumentos CLI
reales con que se lanzó cada modelo), y un chat integrado para
interrogar al servidor que estás vigilando — todo en un solo archivo
HTML que puedes dejar junto a llama-server o abrir directamente desde
GitHub Pages.

## Qué incluye

- **Vista en tiempo real** — velocidad de generación, cola de
  peticiones y actividad de slots actualizadas en directo.
- **Chat integrado** — habla con tu modelo desde el propio panel.
  Respuestas en streaming, renderizado de Markdown, botón para
  detener y sliders de temperature / top_p / max_tokens.
- **Consejos en lenguaje natural** — cuando el servidor está bajo
  presión, te dice qué opción ajustar (por ejemplo: "la cola está
  creciendo, prueba a aumentar `--parallel` en 1").
- **Detalle por slot** — toda la configuración de muestreo
  (temperature, top_k, top_p, repeat_penalty, mirostat, …) reunida
  en un solo sitio.
- **Interfaz en cinco idiomas** — inglés / coreano / japonés /
  chino simplificado / español. Se selecciona automáticamente según
  tu navegador y puedes cambiarla en cualquier momento desde la
  cabecera.
- **Detección automática** de servidores single-model y multi-modelo
  (router).

## Cómo empezar

Necesitas un llama-server razonablemente reciente, ejecutado con
`--metrics`. Lo más sencillo es dejar que el propio llama-server
sirva también `monitor.html`.

```bash
mkdir -p public
cp monitor.html public/
llama-server -m model.gguf --metrics --port 8080 --path ./public
```

Después abre `http://localhost:8080/monitor.html` en el navegador.

Para monitorizar un servidor en otra máquina, aloja `monitor.html` en
cualquier hosting estático y pásale el parámetro `?server=`.

```
http://localhost:8000/monitor.html?server=http://10.0.0.5:8080
```

## Opciones de URL

| Parámetro | Descripción |
|---|---|
| `server` | URL de llama-server (por defecto: mismo origen) |
| `model` | modelo preseleccionado en modo router |
| `lang` | idioma de la interfaz (`en` / `ko` / `ja` / `zh-CN` / `es`) |
| `demo` | `1` o `router` para usar el servidor mock integrado |

Los ajustes se guardan únicamente en la URL: comparte un enlace y
quien lo abra verá exactamente la misma vista.

## Opcional: log del servidor

El panel hace tail de `server.log` automáticamente cuando puede
encontrar uno en el mismo origen que `monitor.html`. Para activarlo,
redirige stdout/stderr de llama-server al directorio que sirves de
forma estática:

```bash
llama-server -m model.gguf --metrics --port 8080 --path ./public \
  > ./public/server.log 2>&1
```

Si no hay archivo de log accesible, el panel se oculta solo, así que
no hace falta configurar nada para que el resto funcione. Usa
`?log=ruta/al/archivo` para cambiar la ruta, o `?log=` (valor vacío)
para desactivarlo explícitamente.

## Privacidad y seguridad

- El panel no envía datos a ningún sitio que no sea la URL de
  llama-server que indiques (y, opcionalmente, un archivo de log en
  el mismo origen).
- No usa cookies, localStorage ni ningún tipo de seguimiento.
- Cualquier acción que modifique el estado del servidor (cargar o
  descargar un modelo, guardar / restaurar / borrar el KV de un slot)
  requiere confirmación previa.
- El renderizado de Markdown es seguro: nada de lo que el modelo
  escriba puede ejecutarse como HTML ni como script.

## Licencia

[MIT](LICENSE). Inspirado en
[abhiFSD/llama.cpp-Monitor-Dashboard](https://github.com/abhiFSD/llama.cpp-Monitor-Dashboard).
¿Quieres contribuir? Consulta [CLAUDE.md](CLAUDE.md).
