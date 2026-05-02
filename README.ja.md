# zerollama-dashboard

[llama.cpp](https://github.com/ggml-org/llama.cpp) サーバ向けの、
HTML 1 ファイルだけで動くモニタリングダッシュボードです。各スロットの
リクエスト状態とサンプリングパラメータをリアルタイムに表示し、調整が
必要なときはどのオプションをどう変えるべきかを自然な文章で示します。
同じページに内蔵されたチャットパネルからプロンプトを送り、メトリクスが
どう反応するかを並べて観察できます。

[**▶ ライブデモを開く**](https://jungrok5.github.io/zerollama-dashboard/?demo=1) — 内蔵モックサーバ上で動くため、実サーバは不要です。

> Languages: [English](README.md) · [한국어](README.ko.md) · **日本語** · [简体中文](README.zh-CN.md) · [Español](README.es.md)

![ダッシュボード](docs/screenshots/main-en.png)
![チャットパネル](docs/screenshots/chat-panel.png)

## このプロジェクトの位置づけ

llama.cpp 公式の WebUI はチャットクライアントで、ライブな
サーバ状態は見られません。Prometheus + Grafana は数値は出せても
「どのフラグをどう変えるか」までは教えてくれず、Open WebUI や
LibreChat などはスロット単位の可視化を持たないチャット専用フロント
エンドです。zerollama-dashboard はその隙間を埋めます: `/metrics` +
`/slots` を運用者視点で表示し、処方箋的な調整ヒントを添え、マルチ
モデル**ルーター**モードにも対応します(各モデルの実際の起動引数も
表示)。同じページのチャットパネルから、監視対象のサーバを直接叩く
こともできます ── すべてが HTML 1 枚に収まっているので、サーバの隣に
置いて開いても、GitHub Pages から直接開いても動きます。

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
- **シングルモデルサーバとマルチモデル(ルーター)サーバ**の両方を自動で
  判別します。

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
| `demo` | `1` または `router` — 内蔵モックサーバを使用 |

設定はすべて URL に含まれるため、リンクを共有すれば同じ画面をそのまま
再現できます。

## オプション機能: サーバログ

ダッシュボードは `monitor.html` と同一 origin に `server.log` を見つけ
られれば、自動的に tail して表示します。有効にするには llama-server の
stdout/stderr を、配信ディレクトリ内のファイルへリダイレクトしてください。

```bash
llama-server -m model.gguf --metrics --port 8080 --path ./public \
  > ./public/server.log 2>&1
```

ログファイルが存在しない場合、パネルは自動的に非表示となるため、
リダイレクトしなくてもダッシュボード自体は問題なく動作します。パスを
変更するには `?log=path/to/file`、明示的に無効化するには `?log=`(空値)
を指定してください。

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
