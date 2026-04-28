# デザイン思考RPG

デザイン思考の5フェーズ（共感・定義・アイデア・実装・テスト）を体験できる教育ゲームです。  
AI NPCとの対話、プレイ後アンケート、CSVデータ収集機能を備えた研究用プロトタイプです。

## システム構成

```
ブラウザ（GitHub Pages）
　　↓ POST /api/chat
Cloudflare Workers（APIプロキシ）
　　↓ x-api-key ヘッダー
Anthropic API（Claude）
```

---

## デプロイ手順

### ステップ 1 — Anthropic APIキーを取得

1. [https://console.anthropic.com](https://console.anthropic.com) にアクセス
2. 「API Keys」→「Create Key」でAPIキーを発行
3. キーをメモしておく（`sk-ant-...` で始まる文字列）

---

### ステップ 2 — Cloudflare Workerをデプロイ

#### 2-1. Cloudflareアカウントを作成

[https://dash.cloudflare.com/sign-up](https://dash.cloudflare.com/sign-up) で無料登録（クレジットカード不要）

#### 2-2. Wranglerをインストール

```bash
npm install -g wrangler
```

#### 2-3. Cloudflareにログイン

```bash
wrangler login
```

ブラウザが開くので認証する。

#### 2-4. Workerをデプロイ

```bash
cd worker
wrangler deploy
```

デプロイ成功すると以下のようなURLが表示される：

```
https://dt-rpg-proxy.your-name.workers.dev
```

このURLをメモしておく。

#### 2-5. APIキーを環境変数として設定

```bash
wrangler secret put ANTHROPIC_API_KEY
```

プロンプトが表示されたら `sk-ant-...` のAPIキーを貼り付けてEnter。

#### 2-6. 動作確認

```bash
curl -X POST https://dt-rpg-proxy.your-name.workers.dev/api/chat \
  -H "Content-Type: application/json" \
  -d '{"system":"あなたは田中さんです。","user":"こんにちは"}'
```

`{"text":"..."}` が返れば成功。

---

### ステップ 3 — GitHub Pagesをデプロイ

#### 3-1. GitHubリポジトリを作成

1. [https://github.com/new](https://github.com/new) でリポジトリを作成
2. リポジトリ名は `dt-rpg` など任意

#### 3-2. ファイルをプッシュ

```bash
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/あなたのユーザー名/dt-rpg.git
git push -u origin main
```

#### 3-3. GitHub Pagesを有効化

1. GitHubのリポジトリページ →「Settings」タブ
2. 左メニュー「Pages」をクリック
3. Source: 「Deploy from a branch」
4. Branch: `main` / Folder: `/docs`
5. 「Save」をクリック

数分後、以下のURLでアクセスできる：

```
https://あなたのユーザー名.github.io/dt-rpg/
```

---

### ステップ 4 — ゲームにWorker URLを設定

1. ゲームのURLをブラウザで開く
2. 表示される入力欄に `https://dt-rpg-proxy.your-name.workers.dev` を入力
3. 「設定して開始」をクリック

Worker URLはブラウザのlocalStorageに保存されるため、次回以降は入力不要。

---

## ファイル構成

```
dt-rpg/
├── docs/
│   └── index.html        # GitHub Pages用ゲーム本体
├── worker/
│   ├── index.js          # Cloudflare Worker（APIプロキシ）
│   └── wrangler.toml     # Workerの設定ファイル
└── README.md
```

---

## データ収集について

プレイヤーがアンケートを送信するとCSVファイルが自動ダウンロードされます。

### CSVのカラム

| カラム名 | 内容 |
|---|---|
| session_id | セッションID（Unix時刻） |
| play_time | プレイ時間（分:秒） |
| score_empathy | 共感フェーズのスコア（0〜5） |
| score_clarity | 定義フェーズのスコア（0〜3） |
| score_creativity | 発想フェーズのスコア（0〜3） |
| score_action | 実装・テストフェーズのスコア（0〜5） |
| total_pct | 合計スコア（%） |
| q1_useful | 「理解に役立った」リッカート（1〜5） |
| q2_phase_diff | 「フェーズの違いが理解できた」リッカート（1〜5） |
| q3_time_ok | 「プレイ時間は適切」リッカート（1〜5） |
| q4_hardest_phase | 最も難しかったフェーズ（1〜5） |
| q5_feedback | 自由記述 |

### Pythonでの分析例

```python
import pandas as pd
import glob

# 複数のCSVを結合
files = glob.glob('data/dt_rpg_*.csv')
df = pd.concat([pd.read_csv(f) for f in files])

# フェーズ別スコアの平均
print(df[['score_empathy','score_clarity','score_creativity','score_action']].mean())

# 最も難しかったフェーズの分布
phase_names = {1:'共感',2:'定義',3:'アイデア',4:'実装',5:'テスト'}
print(df['q4_hardest_phase'].map(phase_names).value_counts())
```

---

## 無料枠について

| サービス | 無料枠 |
|---|---|
| GitHub Pages | 無制限（パブリックリポジトリ） |
| Cloudflare Workers | 1日10万リクエストまで無料 |
| Anthropic API | 従量課金（1リクエスト約0.003円） |

学会発表規模（数十〜百人程度）であれば実質無料で運用できます。

---

## 論文での記載例

> 本システムはフロントエンドをGitHub Pagesで静的ホスティングし、LLM APIへのアクセスをCloudflare Workersを介したサーバーレスアーキテクチャで管理することで、APIキーの安全性と無償公開を両立した。NPCの応答はClaude（Anthropic）を用いてリアルタイム生成し、プレイごとに異なるインタラクションを実現した。

---

## ライセンス

MIT License — 研究・教育目的での利用・改変を自由に行えます。
