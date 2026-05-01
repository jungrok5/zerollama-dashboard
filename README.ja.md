# zerollama-dashboard

[llama.cpp](https://github.com/ggml-org/llama.cpp) サーバの状態を 1 画面に
表示するリアルタイムダッシュボードです。ブラウザで開けばモデルの動きが
ひと目で分かり、同じ画面からそのモデルとチャットすることもできます。

[**▶ ライブデモを開く**](https://jungrok5.github.io/zerollama-dashboard/?demo=1) — 実際のサーバを用意しなくても画面の様子を確認できます。

> Languages: [English](README.md) · [한국어](README.ko.md) · **日本語** · [简体中文](README.zh-CN.md) · [Español](README.es.md)

![ダッシュボード](docs/screenshots/main-en.png)
![チャットパネル](docs/screenshots/chat-panel.png)
![モバイル表示](docs/screenshots/mobile.png)

## 主な機能

- **リアルタイム監視** — 生成速度、リクエストキュー、スロットの稼働状況が
  リアルタイムに更新されます。
- **チャットを内蔵** — 同じ画面でモデルと対話できます。ストリーミング
  応答、Markdown レンダリング、停止ボタン、temperature / top_p /
  max_tokens スライダーを備えています。
- **分かりやすい助言** — サーバに負荷がかかると、どのオプションをどう
  調整すべきかを自然な日本語で提案します(例:「リクエストがキューに
  溜まっています。`--parallel` を 1 増やしてみてください」)。
- **スロットごとの詳細** — サンプリング設定一式(temperature、top_k、
  top_p、repeat_penalty、mirostat など)を 1 か所で確認できます。
- **5 言語対応 UI** — 英語 / 韓国語 / 日本語 / 中国語(簡体) / スペイン語。
  ブラウザの言語で自動起動し、ヘッダからいつでも切り替えられます。
- **シングルモデルとルーターモード**の両方を自動検出します。

## セットアップ

比較的新しい llama-server に `--metrics` オプションを付けて起動するだけ
です。最も簡単な方法は、llama-server 自身に `monitor.html` も配信させる
ことです。

```bash
mkdir -p public
cp monitor.html public/
llama-server -m model.gguf --metrics --port 8080 --path ./public
```

その後、ブラウザで `http://localhost:8080/monitor.html` にアクセスします。

別のマシンで動いているサーバを監視する場合は、`monitor.html` を任意の
静的ホスティングに置き、`?server=` パラメータで対象サーバを指定します。

```
http://localhost:8000/monitor.html?server=http://10.0.0.5:8080
```

## URL オプション

| パラメータ | 説明 |
|---|---|
| `server` | llama-server の URL(既定: 同一 origin) |
| `model` | ルーターモードで既定選択するモデル |
| `lang` | UI 言語(`en` / `ko` / `ja` / `zh-CN` / `es`) |
| `prompt` | チャット入力欄に初期値を設定 |
| `demo` | `1` または `router` — 内蔵モックサーバを使用 |
| `poll` | ポーリング間隔、ms(既定 `1000`) |
| `log` | ログファイルパス(省略時は自動検出) |

設定はすべて URL に含まれるため、リンクを共有すれば同じ画面をそのまま
再現できます。

## 自分でデモを公開する

誰でもアクセスできる公開 URL が必要な場合は、GitHub Pages が無料で
idle sleep もないため適しています。

1. リポジトリの **Settings → Pages → Source = "GitHub Actions"** を設定
2. `main` ブランチに push

同梱の `.github/workflows/pages.yml` が自動的にビルドおよびデプロイを
行います。訪問者は `https://<ユーザ>.github.io/<repo>/?demo=1` から
アクセスできます。すべて静的ファイルのため、維持すべきバックエンドは
ありません。

## プライバシーと安全性

- ダッシュボードはユーザーが指定した llama-server の URL(および同一
  origin にあるログファイル)以外にはデータを送信しません。
- Cookie、localStorage、トラッキングは一切使用しません。
- サーバの状態を変更する操作(モデルの load / unload、スロット KV の
  save / restore / erase)は、実行前に必ず確認ダイアログを表示します。
- チャットの Markdown レンダリングは安全に処理されます。モデルの応答
  内容が HTML やスクリプトとして実行されることはありません。

## ライセンス

[MIT](LICENSE). [abhiFSD/llama.cpp-Monitor-Dashboard](https://github.com/abhiFSD/llama.cpp-Monitor-Dashboard) にインスパイアされています。
コントリビュート方法は [CLAUDE.md](CLAUDE.md) を参照してください。
