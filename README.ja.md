# zerollama-dashboard

[llama.cpp](https://github.com/ggml-org/llama.cpp) サーバ用の、
依存ゼロ・単一 HTML ファイルのモニタリングダッシュボード。

> Languages: [English](README.md) · [한국어](README.ko.md) · **日本語** · [简体中文](README.zh-CN.md) · [Español](README.es.md)

> [abhiFSD/llama.cpp-Monitor-Dashboard](https://github.com/abhiFSD/llama.cpp-Monitor-Dashboard) (MIT) にインスパイアされています。
> npm なし、CDN なし、localStorage なし。HTML ファイル一つだけ。

## 表示する情報

- リアルタイム `/metrics` (Prometheus): 生成/プロンプト tok/s、KV キャッシュ
  使用率、処理中/待機中リクエスト、累計カウンタ
- `/slots`: スロットごとの状態と全サンプリングパラメータ
  (temperature, top_k, top_p, min_p, repeat_penalty, mirostat, DRY など)
- `/props` + `/v1/models`: モデルメタデータ (vocab, context, embedding
  次元、パラメータ数、chat template、modalities、ビルド情報)
- `/lora-adapters`: ロード済み LoRA アダプタとスケール
- `/models` (ルーターモード): キャッシュ内全モデルとステータス (loaded /
  loading / unloaded / sleeping / failed) + **各モデルの実際の起動 CLI 引数**
- オプション `server.log` の tail (HTTP Range、自動検出; 利用不可なら自動で隠蔽)
- **インラインガイダンス**: 各カードの ⓘ ツールチップでパラメータを説明し、
  状態が閾値を超えると推奨アクションを表示 (例: 「KV キャッシュ 96% — 
  `--ctx-size` を増やすか `--parallel` を減らすことを検討」)

## クイックスタート

### オプション A — llama-server と同一 origin

```bash
mkdir -p ./public
cp monitor.html ./public/
llama-server -m model.gguf --metrics --port 8080 --path ./public
# http://localhost:8080/monitor.html を開く
```

### オプション B — リモートサーバを指定

`monitor.html` を任意の静的サーバ (例: `python3 -m http.server`) で配信し、
`?server=` を渡します:

```
http://localhost:8000/monitor.html?server=http://10.0.0.5:8080
```

リモートの `llama-server` は CORS を許可している必要があります (デフォルトは寛容)。

### ルーター (マルチモデル) モード

`-m` を **付けずに** llama-server を起動:

```bash
llama-server --models-dir ./models --metrics --port 8080
```

`GET /models` を叩いて自動的にルーターモードを検出します。ヘッダーにモデル
セレクタが表示されます。

## URL パラメータ

| Param | デフォルト | 用途 |
|---|---|---|
| `server` | same origin | llama-server のベース URL |
| `model` | (なし) | ルーターモード: 既定で選択するモデル |
| `poll` | `1000` | ポーリング間隔 (ms) |
| `log` | auto | ログファイルパス; 未指定なら自動検出、到達不能ならパネル非表示 |
| `lang` | auto | `en` / `ko` / `ja` / `zh-CN` / `es` (ブラウザのデフォルト言語) |

設定はすべて URL に保持 — localStorage は使いません。リンクを共有 = 同じビュー。

## オプション: ログ tail

ログパネルは `Range: bytes=-65536` で静的ファイルの末尾 ~64 KB を読み取ります。
動作には **3 つすべて** が必要です:

1. llama-server の stdout/stderr がファイルにリダイレクトされていること:
   ```bash
   llama-server ... --path ./public > ./public/server.log 2>&1
   ```
   (systemd / docker は既定で stdout のみ — ファイルは生成されません)
2. そのファイルが `monitor.html` と同じ origin から配信されていること。
3. HTTP サーバが `Range` をサポートしていること (cpp-httplib・nginx は対応)。

いずれかが満たされない場合、パネルは静かに隠れます。`?log=path/to/file` で
パスを上書き、`?log=` で明示的に無効化できます。

## llama-server 要件

- `/metrics`, `/slots`, `/props`, `/v1/models`, `/lora-adapters` を提供する
  十分新しいビルド (いずれも標準)
- `/metrics` を有効化するには `--metrics`
- `--slots` は既定で有効; `--no-slots` は付けないでください
- ルーターモード: `-m` を付けずに、`--models-dir` か `--models-preset` で起動

## ガイダンスルール (抜粋)

| シグナル | しきい値 | 提案 |
|---|---|---|
| KV キャッシュ使用率 | > 90% 継続 | `--ctx-size` を増やす、`--parallel` を減らす、または `--ctx-shift` を有効化 |
| 待機リクエスト | > 0 継続 | `--parallel` を増やす、もしくはクライアントの同時実行を減らす |
| 生成 tok/s が低い + スロットがアイドル | — | `--n-gpu-layers` を増やす |
| `is_sleeping` true | — | 次のリクエストはモデル再ロード — `--sleep-idle-seconds` を調整 |
| スロット `temperature` 0 | — | Greedy デコーディング (決定的) |
| スロット `temperature` > 1.5 | — | 品質低下の恐れ; 通常 ≤1.0 |
| スロット `repeat_penalty` > 1.3 | — | フォーマットを壊す可能性; 通常 1.05–1.15 |
| スロット `mirostat` ≠ 0 | — | top_p / top_k は無視される |
| ルーターモデル `failed` | — | `exit_code` を確認; args / VRAM をチェック |

全ルールセットは条件成立時にダッシュボードの「Active suggestions」パネルに
描画されます。

## ライセンス

[MIT](LICENSE). [abhiFSD/llama.cpp-Monitor-Dashboard](https://github.com/abhiFSD/llama.cpp-Monitor-Dashboard) にインスパイアされています。
