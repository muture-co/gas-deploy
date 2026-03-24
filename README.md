# gas-deploy

HTMLファイルをGoogle Apps ScriptのWebアプリとしてデプロイするCLIツール。

## 何ができるか

[clasp](https://github.com/google/clasp) をラップし、2ステップのシンプルなワークフローを提供する:

1. `gas-deploy init my-app.html` — Apps Scriptプロジェクトを作成し、Webアプリとしてデプロイ。固定URLを発行
2. `gas-deploy push my-app.html` — 同じURLのまま、最新のHTMLでWebアプリを更新

Apps Scriptエディタを開く必要なし。デプロイIDの管理も不要。HTMLをpushするだけ。

### URL固定化

claspの `clasp deploy` を引数なしで実行すると、毎回新しいデプロイIDとURLが生成される。
`gas-deploy` は初回 `init` 時にデプロイIDを `.gas-deploy.json` に保存し、以降の `push` では常に同じデプロイIDを更新するため、**URLが変わらない**。

共有済みのURLがそのまま使い続けられる。

## 前提条件

- **macOS または Linux**（Windows WSLも可）
- **Node.js**（macOSではHomebrewで自動インストール）
- **Googleアカウント**（Apps Script APIが有効であること）

## セットアップ

```bash
# インストール
git clone https://github.com/muture-co/gas-deploy.git ~/source/gas-deploy
mkdir -p ~/bin && ln -s ~/source/gas-deploy/gas-deploy ~/bin/gas-deploy

# ~/bin にPATHを通す（.zshrcに未設定の場合）
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

初回実行時に以下を自動で行う:
- `clasp` が未インストールなら `npm install -g @google/clasp`
- Google OAuthログインをブラウザで開く

## 使い方

```bash
# 初期化（初回のみ — プロジェクト作成＋デプロイ）
gas-deploy init my-dashboard.html --title "ダッシュボード"

# 更新（HTMLを変更したらpushするだけ）
gas-deploy push my-dashboard.html

# 登録済みアプリの一覧
gas-deploy list

# ブラウザで開く
gas-deploy open my-dashboard.html
```

## 仕組み

```
my-app.html
    │
    ▼
gas-deploy init
    │
    ├── clasp create（Apps Scriptプロジェクト作成）
    ├── Code.gs 生成（doGet → HtmlService）
    ├── appsscript.json 生成（Webアプリ設定）
    ├── clasp push（ファイルアップロード）
    ├── clasp deploy（Webアプリ作成）
    └── .gas-deploy.json に設定保存
    │
    ▼
https://script.google.com/macros/s/.../exec  ← 固定URL
```

以降の `gas-deploy push` は同じデプロイIDを更新するので、URLは変わらない。

## コマンド一覧

| コマンド | 説明 |
|---------|------|
| `gas-deploy init <file> --title "名前"` | プロジェクト作成＋初回デプロイ |
| `gas-deploy push <file>` | 既存デプロイを更新（URL固定） |
| `gas-deploy list` | 登録済みアプリの一覧表示 |
| `gas-deploy open <file>` | デプロイURLをブラウザで開く |

### `init` のオプション

| オプション | デフォルト | 説明 |
|-----------|----------|------|
| `--title "名前"` | ファイル名 | ブラウザタブに表示されるタイトル |
| `--access DOMAIN` | DOMAIN | `DOMAIN` = 組織内のみ、`ANYONE_ANONYMOUS` = 一般公開 |

## 設定ファイル

`gas-deploy` はカレントディレクトリの `.gas-deploy.json` に設定を保存する:

```json
{
  "apps": {
    "my-app.html": {
      "script_id": "1vNzsvh...",
      "deploy_id": "AKfycbx...",
      "title": "My App",
      "access": "DOMAIN",
      "gas_dir": "/tmp/gas-deploy-my-app"
    }
  }
}
```

1つのディレクトリで複数のHTMLファイルを管理できる。それぞれ独立したApps Scriptプロジェクトになる。

## Claude Codeとの連携

ビルド/同期スクリプトと組み合わせて自動デプロイ:

```bash
#!/bin/bash
# sync_my_app.sh

# Step 1: データ加工
python3 build_data.py > data.json
python3 inject_data.py template.html data.json > my-app.html

# Step 2: デプロイ
gas-deploy push my-app.html
```

`CLAUDE.md` に記載しておけば、Claude Codeが自動でデプロイまで実行する:

```markdown
## デプロイ
my-app.html を変更したら:
bash sync_my_app.sh
```

## アクセス制御

- **`DOMAIN`**（デフォルト）: Google Workspace組織内のユーザーのみアクセス可能
- **`ANYONE_ANONYMOUS`**: URLを知っている人なら誰でもアクセス可能（ログイン不要）

init後にアクセス設定を変更するには:
1. `https://script.google.com/d/<script_id>/edit` を開く
2. デプロイ → デプロイを管理 → 編集 → アクセス

## トラブルシューティング

### "User has not enabled the Apps Script API"
→ https://script.google.com/home/usersettings でApps Script APIを有効にする

### clasp loginでブラウザが開くが何も起きない
→ 正しいGoogleアカウントでブラウザにログインしているか確認

### デプロイしたがアクセスできない
→ 初回デプロイ後、Apps Scriptエディタでアクセス設定の手動変更が必要な場合がある（デプロイ → デプロイを管理 → アクセス編集）

### 大きなHTMLファイル（50MB超）
→ Apps Scriptには1ファイル50MBの制限がある。分割するかアセットを最適化する

## ライセンス

MIT
