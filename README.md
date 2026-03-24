# gas-deploy

社内限定のWebページをワンコマンドで公開するCLIツール。

HTMLファイルをGoogle Apps ScriptのWebアプリとしてデプロイする。Google Workspaceの認証基盤をそのまま使えるので、社内メンバーだけがアクセスできるページを簡単に作れる。

## ユースケース

- 社内向けダッシュボード
- チーム限定のツール・ビューア
- プロトタイプの社内共有
- スプレッドシートのデータを表示するページ（将来対応予定）

## 特徴

- **URL固定**: 何度更新しても同じURLで公開される。共有リンクが壊れない
- **社内限定**: Google Workspaceのドメイン認証で、組織外からはアクセス不可（デフォルト）
- **ワンコマンド更新**: `gas-deploy push` だけ。Apps Scriptエディタを開く必要なし

### なぜURL固定が必要か

Google Apps Scriptの `clasp deploy` は、引数なしで実行すると毎回新しいデプロイIDとURLが生成される。チームに共有したURLが次のデプロイで無効になってしまう。

`gas-deploy` は初回 `init` 時にデプロイIDを `.gas-deploy.json` に保存し、以降の `push` では常に同じデプロイIDを更新する。URLは一度共有すればずっと使える。

## 前提条件

- macOS または Linux（Windows WSLも可）
- Node.js（macOSではHomebrewで自動インストール）
- Google Workspaceアカウント（Apps Script APIが有効であること）

## セットアップ

```bash
# インストール
git clone https://github.com/muture-co/gas-deploy.git ~/source/gas-deploy
mkdir -p ~/bin && ln -s ~/source/gas-deploy/gas-deploy ~/bin/gas-deploy

# PATHを通す（.zshrcに未設定の場合）
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

初回実行時に以下を自動で行う:
- `clasp` が未インストールなら自動インストール
- Google OAuthログインをブラウザで開く

## 使い方

### 1. 初期化（初回のみ）

```bash
gas-deploy init my-page.html --title "社内ダッシュボード"
```

Apps Scriptプロジェクトが作成され、Webアプリとしてデプロイされる。
完了すると、アクセスURL・エディタURLが表示される:

```
✓ デプロイ完了

  URL:        https://script.google.com/macros/s/AKfycbx.../exec
  エディタ:   https://script.google.com/d/1vNzsvh.../edit

次のステップ: 権限設定
  初回デプロイ後、Apps Scriptエディタで権限を設定してください:
  1. 上記エディタURLを開く
  2. 「デプロイ」→「デプロイを管理」
  3. 編集（鉛筆アイコン）→「アクセスできるユーザー」を確認
  4.「次のユーザーとして実行」→「自分」を選択
```

### 2. 初回の権限設定（重要）

初回デプロイ後、Webアプリにアクセスするには Apps Script ポータル上で権限設定が必要。

#### 手順

1. **エディタを開く**
   `gas-deploy init` 完了時に表示されるエディタURLを開く

2. **「デプロイ」→「デプロイを管理」を選択**
   画面右上の「デプロイ」ボタンをクリック

3. **編集（鉛筆アイコン）をクリック**

4. **以下を確認・設定**:
   | 項目 | 設定値 |
   |------|--------|
   | 次のユーザーとして実行 | **自分** |
   | アクセスできるユーザー | **組織内の全員**（社内限定の場合） |

5. **「デプロイ」をクリック**

6. **初回アクセス時の承認**
   URLに初めてアクセスすると「承認が必要です」と表示される。
   「権限を確認」→ Googleアカウントを選択 → 「許可」をクリック。

> この権限設定は初回のみ。以降の `gas-deploy push` ではURLも権限もそのまま維持される。

### 3. 更新

HTMLを変更したら:

```bash
gas-deploy push my-page.html
```

同じURLで最新の内容に更新される:

```
✓ 更新完了 → https://script.google.com/macros/s/AKfycbx.../exec
```

### 4. その他のコマンド

```bash
# 登録済みアプリの一覧（URLも表示）
gas-deploy list

# デプロイURLをブラウザで開く
gas-deploy open my-page.html
```

## 仕組み

```
my-page.html
    │
    ▼
gas-deploy init
    ├── clasp create（Apps Scriptプロジェクト作成）
    ├── Code.gs 生成（doGet → HtmlService）
    ├── appsscript.json 生成（Webアプリ設定）
    ├── clasp push → clasp deploy
    └── .gas-deploy.json にデプロイID保存
    │
    ▼
https://script.google.com/macros/s/.../exec  ← この URL は固定

gas-deploy push
    ├── clasp push（ファイル更新）
    └── clasp deploy --deploymentId（同じIDを更新）
    │
    ▼
同じ URL で最新の内容が反映される
```

## 設定ファイル

`gas-deploy` はカレントディレクトリの `.gas-deploy.json` に設定を保存する:

```json
{
  "apps": {
    "my-page.html": {
      "script_id": "1vNzsvh...",
      "deploy_id": "AKfycbx...",
      "title": "社内ダッシュボード",
      "access": "DOMAIN",
      "gas_dir": "/tmp/gas-deploy-my-page"
    }
  }
}
```

1つのディレクトリで複数のHTMLファイルを管理できる。それぞれ独立したApps Scriptプロジェクトになる。

## コマンド一覧

| コマンド | 説明 |
|---------|------|
| `gas-deploy init <file> --title "名前"` | プロジェクト作成＋初回デプロイ |
| `gas-deploy push <file>` | 既存デプロイを更新（URL固定） |
| `gas-deploy list` | 登録済みアプリの一覧表示（URL付き） |
| `gas-deploy open <file>` | デプロイURLをブラウザで開く |

### `init` のオプション

| オプション | デフォルト | 説明 |
|-----------|----------|------|
| `--title "名前"` | ファイル名 | ブラウザタブに表示されるタイトル |
| `--access DOMAIN` | DOMAIN | `DOMAIN` = 組織内のみ、`ANYONE_ANONYMOUS` = 一般公開 |

## Claude Codeとの連携

ビルドスクリプトに `gas-deploy push` を組み込めば、Claude Codeがファイル変更からデプロイまで一気通貫で実行できる:

```bash
#!/bin/bash
# sync_my_app.sh

# データ加工
python3 build_data.py > data.json
python3 inject_data.py template.html data.json > my-page.html

# デプロイ
gas-deploy push my-page.html
```

`CLAUDE.md` に記載しておく:

```markdown
## デプロイ
my-page.html を変更したら `bash sync_my_app.sh` を実行すること。
```

## トラブルシューティング

### "User has not enabled the Apps Script API"
→ https://script.google.com/home/usersettings でApps Script APIを有効にする

### clasp loginでブラウザが開くが何も起きない
→ 正しいGoogleアカウントでブラウザにログインしているか確認

### デプロイ後にURLを開くと「アクセスが拒否されました」
→ 「初回の権限設定」セクションの手順を実施する。特に「次のユーザーとして実行」が「自分」になっているか確認

### 大きなHTMLファイル（50MB超）
→ Apps Scriptには1ファイル50MBの制限がある

## TODO

- [ ] スプレッドシート連携（HTMLからスプレッドシートのデータを読み書き）
- [ ] `gas-deploy status` コマンド（デプロイ状態の確認）
- [ ] テンプレート機能（よく使うHTML構成のひな形）

## ライセンス

MIT
