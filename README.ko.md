# zerollama-dashboard

[llama.cpp](https://github.com/ggml-org/llama.cpp) 서버 상태를 한 페이지에
보여주는 실시간 대시보드입니다. 브라우저에서 열기만 하면 모델이 무엇을
하고 있는지 한눈에 확인할 수 있고, 같은 화면에서 모델과 대화도 가능합니다.

[**▶ 라이브 데모 열기**](https://jungrok5.github.io/zerollama-dashboard/?demo=1) — 실제 서버 없이 화면 구성을 먼저 확인할 수 있습니다.

> Languages: [English](README.md) · **한국어** · [日本語](README.ja.md) · [简体中文](README.zh-CN.md) · [Español](README.es.md)

![대시보드](docs/screenshots/main-en.png)
![채팅 패널](docs/screenshots/chat-panel.png)

## 주요 기능

- **실시간 모니터링** — 생성 속도, 요청 대기열, 슬롯 활동이 실시간으로
  갱신됩니다.
- **내장 채팅** — 같은 페이지에서 모델과 대화할 수 있습니다. 스트리밍
  응답, 마크다운 렌더링, 중지 버튼, temperature / top_p / max_tokens
  슬라이더를 제공합니다.
- **이해하기 쉬운 권장사항** — 서버에 부하가 걸리면 어떤 옵션을 어떻게
  조정해야 할지 자연어로 안내합니다. 예: "요청이 대기열에 쌓이고
  있습니다. `--parallel`을 1 늘려 보세요."
- **슬롯별 상세 정보** — 모든 샘플링 설정(temperature, top_k, top_p,
  repeat_penalty, mirostat 등)을 한곳에서 확인할 수 있습니다.
- **5개 언어 UI** — 한국어 / 영어 / 일본어 / 중국어 / 스페인어. 브라우저
  언어로 자동 시작되며, 헤더에서 언제든 변경할 수 있습니다.
- **단일 모델 서버와 멀티 모델(라우터) 서버**를 자동으로 감지합니다.

## 시작하기

비교적 최신 버전의 llama-server에 `--metrics` 옵션만 추가하면 됩니다.
가장 간단한 방법은 llama-server가 `monitor.html`도 함께 제공하도록 두는
것입니다.

```bash
mkdir -p public
cp monitor.html public/
llama-server -m model.gguf --metrics --port 8080 --path ./public
```

이후 브라우저에서 `http://localhost:8080/monitor.html`로 접속합니다.

다른 컴퓨터에 있는 서버를 모니터링하려면 `monitor.html`을 정적 호스팅
어디든 올린 뒤 `?server=` 파라미터로 대상 서버를 지정합니다.

```
http://localhost:8000/monitor.html?server=http://10.0.0.5:8080
```

## URL 옵션

| 파라미터 | 설명 |
|---|---|
| `server` | llama-server URL (기본값: 같은 origin) |
| `model` | 라우터 모드에서 기본으로 선택할 모델 |
| `lang` | UI 언어 (`en` / `ko` / `ja` / `zh-CN` / `es`) |
| `prompt` | 채팅 입력란 초기 텍스트 |
| `demo` | `1` 또는 `router` — 내장 mock 서버 사용 |
| `poll` | 폴링 간격, ms (기본값 `1000`) |
| `log` | 로그 파일 경로 (생략 시 자동 감지) |

설정값은 모두 URL에 담깁니다. 링크 하나로 동일한 화면을 그대로 공유할
수 있습니다.

## 개인정보와 안전성

- 대시보드는 사용자가 지정한 llama-server URL(과 선택적으로 같은
  origin의 로그 파일 경로) 외에는 어떤 곳에도 데이터를 전송하지
  않습니다.
- 쿠키, localStorage, 트래킹은 일절 사용하지 않습니다.
- 서버 상태를 변경하는 작업(모델 load/unload, 슬롯 KV
  save/restore/erase)은 모두 확인 대화상자를 거칩니다.
- 채팅의 마크다운 렌더링은 안전하게 처리됩니다. 모델 응답이 무엇이든
  HTML이나 스크립트로 실행되지 않습니다.

## 라이선스

[MIT](LICENSE). [abhiFSD/llama.cpp-Monitor-Dashboard](https://github.com/abhiFSD/llama.cpp-Monitor-Dashboard)에서 영감을 받아 만들었습니다.
기여 방법은 [CLAUDE.md](CLAUDE.md)를 참고하세요.
