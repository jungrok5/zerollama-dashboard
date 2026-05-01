# zerollama-dashboard

[llama.cpp](https://github.com/ggml-org/llama.cpp) 서버의 현재 상태를
한 페이지에 보여주는 라이브 대시보드. 브라우저로 열고, 모델이 무얼
하고 있는지 보고, 같은 화면에서 모델과 채팅도 할 수 있어요.

[**▶ 라이브 데모 보기**](https://jungrok5.github.io/zerollama-dashboard/?demo=1) — 진짜 서버 없이 분위기만 먼저 확인할 수 있어요.

> Languages: [English](README.md) · **한국어** · [日本語](README.ja.md) · [简体中文](README.zh-CN.md) · [Español](README.es.md)

![대시보드](docs/screenshots/main-en.png)
![채팅 패널](docs/screenshots/chat-panel.png)
![모바일 화면](docs/screenshots/mobile.png)

## 무엇을 볼 수 있나

- **실시간 상태** — 생성 속도, 요청 대기열, 슬롯 활동이 라이브로 갱신
- **모델과 채팅**을 대시보드 안에서 바로. 스트리밍 응답, 마크다운
  렌더, 중지 버튼, temperature / top_p / max tokens 슬라이더
- **사람말 조언** — 서버에 부담이 갈 때 그 자리에서 알려줌
  (예: "대기열이 쌓이고 있어요, `--parallel`을 +1 해보세요")
- **슬롯별 상세** — 샘플링 설정 전부 (temperature, top_k, top_p,
  repeat_penalty, mirostat …)
- **5개 국어 UI** — 한국어 / 영어 / 일본어 / 중국어 / 스페인어. 브라우저
  언어로 자동 시작하고, 헤더에서 언제든 바꿀 수 있음
- **단일 모델과 라우터 서버** 둘 다 자동으로 감지

## 설치

비교적 최신 llama-server에 `--metrics`만 추가하면 됩니다. 가장 간단한
구성은 llama-server가 직접 `monitor.html`을 서빙하는 거예요:

```bash
mkdir -p public
cp monitor.html public/
llama-server -m model.gguf --metrics --port 8080 --path ./public
```

이후 `http://localhost:8080/monitor.html`에 접속.

다른 컴퓨터의 서버에 붙이려면 `monitor.html`을 아무 정적 호스팅에 올리고
`?server=`로 가리키기:

```
http://localhost:8000/monitor.html?server=http://10.0.0.5:8080
```

## URL 옵션

| 파라미터 | 의미 |
|---|---|
| `server` | llama-server URL (기본: 같은 origin) |
| `model` | 라우터 모드: 기본 선택 모델 |
| `lang` | UI 언어 (`en` / `ko` / `ja` / `zh-CN` / `es`) |
| `prompt` | 채팅 입력칸 미리 채움 |
| `demo` | `1` 또는 `router` — 내장 mock 서버 |
| `poll` | 폴링 간격, ms (기본 `1000`) |
| `log` | 로그 파일 경로 (생략 시 자동 감지) |

설정은 전부 URL 안에. 링크 하나로 같은 화면을 그대로 공유할 수 있어요.

## 직접 데모 호스팅하기

누구나 열 수 있는 공개 URL이 필요하면 GitHub Pages가 무료이고 idle
sleep도 없어요:

1. 레포 **Settings → Pages → Source = "GitHub Actions"**
2. `main`에 push

레포에 포함된 `.github/workflows/pages.yml`이 자동으로 빌드·배포합니다.
방문자는 `https://<유저>.github.io/<레포>/?demo=1`로 접속.
정적 파일이라 유지할 백엔드가 없어요.

## 프라이버시와 안전

- 대시보드는 여러분이 지정한 llama-server URL(과 같은 origin의 로그 파일
  경로 외에는) 어디에도 데이터를 보내지 않아요.
- 쿠키, localStorage, 트래킹 같은 것 일체 사용 안 함.
- 서버 상태를 바꾸는 작업 — 모델 load/unload, 슬롯 KV
  save/restore/erase — 은 모두 confirm 다이얼로그를 거칩니다.
- 채팅의 마크다운 렌더는 안전. 모델이 무엇을 출력해도 HTML이나
  스크립트로 빠져나갈 수 없습니다.

## 라이선스

[MIT](LICENSE). [abhiFSD/llama.cpp-Monitor-Dashboard](https://github.com/abhiFSD/llama.cpp-Monitor-Dashboard)에서 영감을 받음.
기여하려면 [CLAUDE.md](CLAUDE.md) 참고.
