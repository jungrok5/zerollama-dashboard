# zerollama-dashboard

为 [llama.cpp](https://github.com/ggml-org/llama.cpp) 服务器打造的实时
状态面板。在浏览器中打开即可查看模型当前的运行情况,并在同一页面中直接
与模型对话。

[**▶ 打开在线演示**](https://jungrok5.github.io/zerollama-dashboard/?demo=1) —— 无需真实服务器即可体验完整界面。

> Languages: [English](README.md) · [한국어](README.ko.md) · [日本語](README.ja.md) · **简体中文** · [Español](README.es.md)

![仪表盘](docs/screenshots/main-en.png)
![聊天面板](docs/screenshots/chat-panel.png)
![移动端视图](docs/screenshots/mobile.png)

## 主要功能

- **实时监控** —— 生成速度、请求队列、槽位活动实时刷新。
- **内置聊天** —— 在同一页面与模型对话,支持流式响应、Markdown 渲染、
  停止按钮,以及 temperature / top_p / max_tokens 滑块。
- **通俗易懂的建议** —— 当服务器压力较大时,以自然语言提示应当如何调整
  参数(例如:"请求正在队列中堆积,可以尝试将 `--parallel` 增加 1")。
- **槽位级别的详细信息** —— 完整的采样配置(temperature、top_k、top_p、
  repeat_penalty、mirostat 等)集中展示。
- **5 种 UI 语言** —— 英文 / 韩文 / 日文 / 简体中文 / 西班牙文。根据
  浏览器语言自动选择,可在页眉随时切换。
- **自动识别**单模型服务器与多模型(路由)服务器。

## 安装

只需要较新版本的 llama-server 并以 `--metrics` 选项启动。最简单的方式
是让 llama-server 同时提供 `monitor.html`。

```bash
mkdir -p public
cp monitor.html public/
llama-server -m model.gguf --metrics --port 8080 --path ./public
```

之后在浏览器中打开 `http://localhost:8080/monitor.html` 即可。

如果要监控其他机器上的服务器,可以将 `monitor.html` 部署到任意静态
托管平台,并通过 `?server=` 参数指定目标服务器。

```
http://localhost:8000/monitor.html?server=http://10.0.0.5:8080
```

## URL 参数

| 参数 | 说明 |
|---|---|
| `server` | llama-server 的 URL(默认:同源) |
| `model` | 路由模式下默认选中的模型 |
| `lang` | UI 语言(`en` / `ko` / `ja` / `zh-CN` / `es`) |
| `prompt` | 聊天输入框的初始内容 |
| `demo` | `1` 或 `router`,使用内置 mock 服务器 |
| `poll` | 轮询间隔,ms(默认 `1000`) |
| `log` | 日志文件路径(省略时自动检测) |

所有设置均保存在 URL 中,因此分享一条链接即可还原相同的视图。

## 隐私与安全

- 仪表盘仅与您指定的 llama-server URL(以及可选的同源日志文件路径)
  通信,不会向任何其他位置发送数据。
- 不使用 cookie、localStorage 或任何形式的追踪。
- 任何会改变服务器状态的操作(加载 / 卸载模型,保存 / 恢复 / 清除
  槽位 KV)都会先弹出确认对话框。
- 聊天面板的 Markdown 渲染是安全的:模型输出的任何内容都不会被作为
  HTML 或脚本执行。

## 许可证

[MIT](LICENSE)。灵感来自 [abhiFSD/llama.cpp-Monitor-Dashboard](https://github.com/abhiFSD/llama.cpp-Monitor-Dashboard)。
如需参与贡献,请阅读 [CLAUDE.md](CLAUDE.md)。
