# zerollama-dashboard

A single-HTML monitoring dashboard for [llama.cpp](https://github.com/ggml-org/llama.cpp)
servers. See every slot's request state and sampling parameters in
real time, and get plain-language hints when something needs tuning.
A built-in chat panel lets you fire prompts and watch the metrics
react on the same page.

[**▶ Try the live demo**](https://jungrok5.github.io/zerollama-dashboard/?demo=1) — runs against a built-in mock server, no backend required.

> Languages: **English** · [한국어](README.ko.md) · [日本語](README.ja.md) · [简体中文](README.zh-CN.md) · [Español](README.es.md)

![dashboard](docs/screenshots/main-en.png)
![chat panel](docs/screenshots/chat-panel.png)

## Why this exists

llama.cpp's built-in WebUI is a chat client and shows no live server
state. Prometheus + Grafana setups expose numbers but won't tell you
which flag to change. Open WebUI, LibreChat and friends are chat
front-ends, not slot-aware monitors. zerollama-dashboard sits in that
gap: a focused operator view of `/metrics` + `/slots` with prescriptive
tuning hints, multi-model **router**-mode awareness (including each
model's actual launch arguments), and a chat panel for poking the
server you're watching — all in one HTML file you can drop next to
`monitor.html` or open straight from GitHub Pages.

## What you get

- **Live monitoring** — generation speed, request queue, and slot
  activity update in real time.
- **Built-in chat** — talk to your model from the same page. Streaming
  responses, Markdown rendering, a stop button, and sliders for
  temperature / top_p / max_tokens.
- **Plain-language tips** — when the server is under pressure, the
  dashboard tells you which option to tune (for example, "the queue
  is growing — try raising `--parallel` by 1").
- **Per-slot detail** — the full sampling configuration (temperature,
  top_k, top_p, repeat_penalty, mirostat, …) all in one place.
- **Five UI languages** — English / 한국어 / 日本語 / 简体中文 /
  Español. Auto-selected from your browser, switchable any time from
  the header.
- **Auto-detection** of single-model and multi-model (router) servers.

## Getting started

You'll need a reasonably recent llama-server started with `--metrics`.
The simplest setup is to let llama-server serve `monitor.html` itself:

```bash
mkdir -p public
cp monitor.html public/
llama-server -m model.gguf --metrics --port 8080 --path ./public
```

Then open `http://localhost:8080/monitor.html`.

To monitor a server on another machine, host `monitor.html` on any
static host and point at the server with `?server=`:

```
http://localhost:8000/monitor.html?server=http://10.0.0.5:8080
```

## URL options

| Parameter | What it does |
|---|---|
| `server` | llama-server URL (default: same origin) |
| `model` | router mode: which model to pre-select |
| `lang` | UI language (`en` / `ko` / `ja` / `zh-CN` / `es`) |
| `demo` | `1` or `router` for the built-in mock server |

Every setting lives in the URL, so a link captures the exact view —
share it and the recipient sees the same thing.

## Optional: server log

The dashboard tails `server.log` automatically when it can reach one
on the same origin as `monitor.html`. To enable it, redirect
llama-server's stdout/stderr into the directory you serve from:

```bash
llama-server -m model.gguf --metrics --port 8080 --path ./public \
  > ./public/server.log 2>&1
```

If no log file is reachable the panel hides itself silently — no
configuration needed. Pass `?log=path/to/file` to override the path,
or `?log=` to disable the panel explicitly.

## Privacy and safety

- The dashboard never sends your data anywhere except the
  llama-server URL you give it (and, optionally, a log file path on
  the same origin).
- It doesn't use cookies, localStorage, or any tracking.
- Anything that changes server state — loading or unloading a model,
  saving / restoring / erasing a slot's KV cache — always asks for
  confirmation first.
- The chat renders Markdown safely: nothing the model writes can
  break out as HTML or scripts.

## License

[MIT](LICENSE). Inspired by
[abhiFSD/llama.cpp-Monitor-Dashboard](https://github.com/abhiFSD/llama.cpp-Monitor-Dashboard).
Contributing? See [CLAUDE.md](CLAUDE.md).
