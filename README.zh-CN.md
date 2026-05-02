# zerollama-dashboard

为 [llama.cpp](https://github.com/ggml-org/llama.cpp) 服务器设计的
单文件 HTML 监控面板。实时显示每个槽位的请求状态与采样参数,并在需要
调整时用自然语言提示应当修改哪个参数。同一页面内置的聊天面板可以发送
提示词,直接观察相关指标如何变化。

[**▶ 打开在线演示**](https://jungrok5.github.io/zerollama-dashboard/?demo=1) —— 在内置 mock 服务器上运行,无需真实后端。

> Languages: [English](README.md) · [한국어](README.ko.md) · [日本語](README.ja.md) · **简体中文** · [Español](README.es.md)

![仪表盘](docs/screenshots/main-en.png)
![聊天面板](docs/screenshots/chat-panel.png)

## 项目定位

llama.cpp 官方内置的 WebUI 是聊天客户端,不展示实时服务器状态;
Prometheus + Grafana 方案能输出数值,却不会告诉你应当修改哪个参数;
Open WebUI、LibreChat 等同类工具是聊天前端,缺乏槽位级可见性。
zerollama-dashboard 正好填补这一空白:以运维者视角展示 `/metrics` 与
`/slots`,提供处方式的调优提示,并支持多模型**路由**模式(可显示每个
模型的实际启动参数)。同一页面的聊天面板还可以直接对所监控的服务器
发起请求 —— 一切都装在一个 HTML 文件里,既能放在服务器目录下打开,
也能直接通过 GitHub Pages 访问。

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
| `demo` | `1` 或 `router`,使用内置 mock 服务器 |

所有设置均保存在 URL 中,因此分享一条链接即可还原相同的视图。

## 可选功能:服务器日志

仪表盘会自动在 `monitor.html` 同源位置查找 `server.log`,找到即开始
tail 显示。要启用此功能,请在启动 llama-server 时将 stdout/stderr
重定向到静态目录中的文件:

```bash
llama-server -m model.gguf --metrics --port 8080 --path ./public \
  > ./public/server.log 2>&1
```

如果日志文件不可达,该面板会自动隐藏,因此不做配置也不会影响仪表盘
正常使用。可通过 `?log=路径` 自定义路径,或通过 `?log=`(空值)显式禁用。

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
