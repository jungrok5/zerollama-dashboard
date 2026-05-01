# zerollama-dashboard

为 [llama.cpp](https://github.com/ggml-org/llama.cpp) 服务器提供的实时
状态页面。在浏览器中打开,就能看到模型正在做什么,还能在同一页面里直接
与模型聊天。

[**▶ 打开在线演示**](https://jungrok5.github.io/zerollama-dashboard/?demo=1) — 无需真实服务器即可体验。

> Languages: [English](README.md) · [한국어](README.ko.md) · [日本語](README.ja.md) · **简体中文** · [Español](README.es.md)

![仪表盘](docs/screenshots/main-en.png)
![聊天面板](docs/screenshots/chat-panel.png)
![移动端视图](docs/screenshots/mobile.png)

## 你能得到什么

- **实时视图** — 生成速度、请求队列、槽位活动实时刷新
- **直接与模型聊天**。流式响应、Markdown 渲染、停止按钮,以及
  temperature / top_p / max_tokens 滑块
- **通俗易懂的建议** — 当服务器吃力时给出提示
  (例如:「队列在堆积,试试 `--parallel +1`」)
- **每个槽位的详细信息**,包括完整采样配置 (temperature, top_k,
  top_p, repeat_penalty, mirostat …)
- **五种 UI 语言**:English / 한국어 / 日本語 / 简体中文 / Español。
  根据浏览器自动选择,可在头部随时切换
- **单模型与路由模式服务器**都自动识别

## 安装

需要一个较新版本的 llama-server,启动时加 `--metrics`。最简单的部署是
让 llama-server 同时提供 `monitor.html`:

```bash
mkdir -p public
cp monitor.html public/
llama-server -m model.gguf --metrics --port 8080 --path ./public
```

打开 `http://localhost:8080/monitor.html` 即可。

要监控其他机器上的服务器,把 `monitor.html` 放到任意静态托管,然后传入
`?server=`:

```
http://localhost:8000/monitor.html?server=http://10.0.0.5:8080
```

## URL 参数

| 参数 | 用途 |
|---|---|
| `server` | llama-server URL (默认: 同源) |
| `model` | 路由模式: 默认选中的模型 |
| `lang` | UI 语言 (`en` / `ko` / `ja` / `zh-CN` / `es`) |
| `prompt` | 预填聊天输入框 |
| `demo` | `1` 或 `router` — 启用内置 mock 服务器 |
| `poll` | 轮询间隔, ms (默认 `1000`) |
| `log` | 日志文件路径 (省略时自动检测) |

所有设置都在 URL 里 — 一条链接就能重现同样的视图。

## 自助部署演示

想要任何人都能访问的公开链接?GitHub Pages 免费且不会休眠:

1. 仓库 **Settings → Pages → Source = "GitHub Actions"**
2. push 到 `main`

仓库自带的 `.github/workflows/pages.yml` 会自动构建并部署。
访客可访问 `https://<用户>.github.io/<仓库>/?demo=1`。
全部静态,无需维护后端。

## 隐私与安全

- 仪表盘只与你指定的 llama-server URL(以及同源的日志文件路径)
  通信,不会把数据发到其他地方。
- 不使用 cookie、localStorage 或任何追踪手段。
- 任何会改变服务器状态的操作 — 加载 / 卸载模型、保存 / 恢复 / 清除
  槽位 KV — 都会先弹出确认对话框。
- 聊天的 Markdown 渲染是安全的。模型输出的任何内容都无法以 HTML 或
  脚本的形式逃逸。

## 许可证

[MIT](LICENSE)。灵感来自 [abhiFSD/llama.cpp-Monitor-Dashboard](https://github.com/abhiFSD/llama.cpp-Monitor-Dashboard)。
参与贡献请阅读 [CLAUDE.md](CLAUDE.md)。
