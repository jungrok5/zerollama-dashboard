# zerollama-dashboard

A live status page for your [llama.cpp](https://github.com/ggml-org/llama.cpp)
server. Open it in a browser, see what your model is doing, and chat with
it from the same page.

[**▶ Try the live demo**](https://jungrok5.github.io/zerollama-dashboard/?demo=1) — runs without a real server, just to give you a feel.

> Languages: **English** · [한국어](README.ko.md) · [日本語](README.ja.md) · [简体中文](README.zh-CN.md) · [Español](README.es.md)

![dashboard](docs/screenshots/main-en.png)
![chat panel](docs/screenshots/chat-panel.png)
![mobile view](docs/screenshots/mobile.png)

## What you get

- **Real-time view** — generation speed, request queue, slot activity,
  all updating live.
- **Chat with your model** right inside the dashboard. Streaming
  responses, Markdown rendering, a stop button, and sliders for
  temperature / top_p / max tokens.
- **Plain-language tips** when the server is under pressure — for
  example, "queue is building up, try `--parallel +1`".
- **Per-slot detail** including the full sampling configuration
  (temperature, top_k, top_p, repeat penalty, mirostat, …).
- **Five UI languages**: English / 한국어 / 日本語 / 简体中文 /
  Español. The dashboard picks one from your browser; you can switch
  any time from the header.
- **Single-model and router servers** are both supported and detected
  automatically.

## Setup

You'll need a recent llama-server, started with `--metrics`. The
simplest layout is to let llama-server itself host `monitor.html`:

```bash
mkdir -p public
cp monitor.html public/
llama-server -m model.gguf --metrics --port 8080 --path ./public
```

Then open `http://localhost:8080/monitor.html`.

To monitor a server on another machine, host `monitor.html` anywhere
static and pass `?server=`:

```
http://localhost:8000/monitor.html?server=http://10.0.0.5:8080
```

## URL options

| Parameter | What it does |
|---|---|
| `server` | llama-server URL (default: same origin) |
| `model` | router mode: which model to pre-select |
| `lang` | UI language (`en` / `ko` / `ja` / `zh-CN` / `es`) |
| `prompt` | prefill the chat input on load |
| `demo` | `1` or `router` for the built-in mock server |
| `poll` | polling interval, ms (default `1000`) |
| `log` | log file path (auto-detected when omitted) |

Settings live in the URL only, so a link captures the exact view —
share it and the recipient sees the same thing.

## Host your own demo

Want a public URL anyone can open? GitHub Pages is free and never
sleeps:

1. Repo **Settings → Pages → Source = "GitHub Actions"**.
2. Push to `main`.

The included workflow (`.github/workflows/pages.yml`) builds and
deploys automatically. Visitors land on
`https://<your-user>.github.io/<repo>/?demo=1`. Everything is static,
so there's no backend to keep alive.

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
