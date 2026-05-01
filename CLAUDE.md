# CLAUDE.md — contributor guidance for zerollama-dashboard

This file is for anyone contributing code (human or AI). It describes
the constraints the project keeps to so the dashboard stays a single,
self-contained HTML file. The filename follows the convention used by
[Claude Code](https://claude.com/claude-code), which automatically
reads it as project context.

If you're a user, you don't need to read this — start with the
[README](README.md).

## Absolute constraints (any change violating these will be rejected)

1. **Single file**: the deliverable is `monitor.html` only. No build step,
   no bundler, no package manager.
2. **No external CDN**: no `<script src="https://...">`, no
   `<link rel="stylesheet" href="https://...">`. Charts are drawn in-line
   with SVG (`<polyline>`).
3. **No localStorage / sessionStorage**: all settings live in URL query
   parameters. Browser storage APIs must not be touched.
4. **All fetch calls wrapped in try/catch and surfaced to the user**.
   Silent failures are forbidden — a monitoring tool that lies is worse
   than one that is down.
5. **No `setInterval`** — use `AbortController` + `setTimeout` chain so
   requests cannot stack and in-flight calls can be aborted.
6. **DOM injection of user/server data uses `textContent` only**. The
   only place `innerHTML` is acceptable is for fully self-authored
   strings (e.g. SVG markup built from numeric values).
7. **Attribution preserved**: README and LICENSE must keep the
   "Inspired by abhiFSD/llama.cpp-Monitor-Dashboard (MIT)" notice.

## Code style

- 2-space indentation, vanilla ES2022. No modules, no top-level await
  (an inline `<script>` tag is fine).
- Small functions, side effects isolated. Single global `state` object.
- DOM updates touch only the changed nodes (no full innerHTML wipe of
  containers like the slot grid).
- Comments: WHY only, not WHAT. One-line preferred.
- All user-facing strings go through the `I18N` table — never hard-code
  English in the markup or JS.

## Testing / verification

After any change:
1. Serve locally: `python3 -m http.server 8000`
2. Run `llama-server` in both modes:
   - single: `llama-server -m model.gguf --metrics --port 8080`
   - router: `llama-server --models-dir ./models --metrics --port 8080`
3. Open `http://localhost:8000/monitor.html?server=http://localhost:8080`
4. Manually exercise error paths: kill the server, run with
   `--no-metrics`, run with `--no-slots`, supply a bogus `?server=`.
5. Check at 360 px width (DevTools mobile preview).
6. Confirm POST controls require a confirm dialog.

## Security

- Outbound fetch is restricted to the URLs derived from `server`, `log`
  query params (and the page origin).
- Server responses are inserted via `textContent` only.
- No API key handling — if the user needs auth, they should front the
  server with a same-origin proxy.

## Attribution

README and LICENSE retain:
> Inspired by [abhiFSD/llama.cpp-Monitor-Dashboard](https://github.com/abhiFSD/llama.cpp-Monitor-Dashboard) (MIT)
