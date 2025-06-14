# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# 🤖 Multi-Claude システム

Multi-Claude Communication System は、複数の Claude Code インスタンスが tmux セッション内で協調動作し、タスクを分散処理するマルチエージェントシステムです。

## アーキテクチャ

```
📊 PRESIDENT セッション (1ペイン)
└── PRESIDENT: ユーザー対話・タスク概要伝達

📊 multiagent セッション (4ペイン)
├── boss1: 要件整理・指示書生成・タスク管理
├── worker1: 実行担当者A（進捗共有）
├── worker2: 実行担当者B（進捗共有）
└── worker3: 実行担当者C（進捗共有）

通信フロー: ユーザー → PRESIDENT → boss1 → workers → boss1 → PRESIDENT
```

## あなたの役割

このシステムでは、あなたがどの tmux ペインで実行されているかによって役割が決まります：

- **president:0** → あなたは **PRESIDENT** です
- **multiagent:0.0** → あなたは **boss1** です
- **multiagent:0.1** → あなたは **worker1** です
- **multiagent:0.2** → あなたは **worker2** です
- **multiagent:0.3** → あなたは **worker3** です

各役割の詳細は `.multi-claude/instructions/` ディレクトリの対応するファイルを参照してください：

- PRESIDENT: `.multi-claude/instructions/president_dynamic.md`
- boss1: `.multi-claude/instructions/boss_dynamic.md`
- worker1,2,3: `.multi-claude/instructions/worker_dynamic.md`

## 開発コマンド

### システム操作

```bash
# 起動・終了
multi-claude                               # システム起動
multi-claude --exit                        # 完全終了
multi-claude --dangerously-skip-permissions # 権限確認スキップ起動

# エージェント間通信（両方対応）
./agent-send.sh [エージェント名] "[メッセージ]"
./.multi-claude/bin/agent-send.sh [エージェント名] "[メッセージ]"
./agent-send.sh --list                     # 利用可能エージェント一覧
```

### テスト実行

```bash
cd tests && ./test_claude_detection.sh              # Claude検出テスト
cd tests && ./test_dangerously_skip_permissions.sh  # オプションテスト
cd tests && ./test_terminal_control.sh              # ターミナル制御テスト
```

### デバッグ

```bash
# tmuxセッション管理
tmux list-sessions                    # 全セッション表示
tmux list-panes -t multiagent         # ペイン構成確認
tmux attach-session -t president      # presidentセッションにアタッチ
tmux kill-server                      # 全セッション強制終了

# ログ確認
cat .multi-claude/logs/send_log.txt              # 全送信ログ
grep "boss1" .multi-claude/logs/send_log.txt     # 特定エージェントのログ
ls -la .multi-claude/tmp/worker*_done.txt        # 完了ファイル確認
ls -la .multi-claude/context/worker*_progress.md # 進捗ファイル確認
```

## 改善されたシステム特徴

### 1. 役割分担の最適化

- **PRESIDENT**: ユーザーとの対話に集中し、タスク概要を素早く BOSS に伝達
- **BOSS**: 詳細な要件整理と具体的な指示書生成を担当
- **WORKER**: 進捗を共有しながら効率的に作業を実行

### 2. クリーンなファイル配置

```
.multi-claude/
├── bin/              # 実行スクリプト
│   ├── setup.sh
│   └── agent-send.sh
├── instructions/     # 役割定義・指示書
│   ├── president_dynamic.md
│   ├── boss_dynamic.md
│   ├── worker_dynamic.md
│   ├── boss_task.md      # 動的生成
│   └── worker_task.md    # 動的生成
├── tmp/              # 一時ファイル
│   ├── worker*_done.txt
│   └── worker_ids/
├── logs/             # ログファイル
│   └── send_log.txt
├── context/          # 進捗共有
│   └── worker*_progress.md
└── tasks/            # タスク管理
    ├── current_task.md
    └── completion_report.md
```

### 3. ワーカー間コンテキスト共有

- 各ワーカーが進捗を `.multi-claude/context/worker[番号]_progress.md` に記録
- 作業開始前に他のワーカーの進捗を確認
- 作業の重複を防ぎ、効率的な協調作業を実現

## リリースワークフロー

### 自動リリース（推奨）

```bash
git tag v1.0.X -m "Release: 変更内容"
git push origin v1.0.X

# 15-18秒後に自動的にHomebrewで利用可能
brew update && brew upgrade multi-claude
```

### GitHub Actions 設定

- **必須シークレット**: `HOMEBREW_GITHUB_TOKEN` (homebrew-multi-claude リポジトリへの`repo`権限)
- **自動処理**: tarball 生成、SHA256 計算、Formula 更新

## 技術仕様

### Claude Code 検出優先順位

1. `$HOME/.claude/local/claude` (直接パス)
2. `which claude` (PATH 検索)
3. `command -v claude` (bash ビルトイン)
4. `claude-code`, `claude.code` (バリエーション)

### コマンドラインオプション

- `--exit`: システム完全終了
- `--help`: ヘルプ表示
- `--version`: バージョン情報（現在: v1.1.0）
- `--dangerously-skip-permissions`: 権限確認スキップ

### トラブルシューティング

#### Claude Code が見つからない場合

```bash
# 実行ファイル検索
find "$HOME" -name "claude*" -type f -perm +111 2>/dev/null | grep -E "(bin|\.local|\.claude)"

# PATH追加
export PATH="$HOME/.claude/local:$PATH"
```

#### 初回セットアップ

## Homebrew インストール時、初回実行で必要なファイルを自動コピー。既存の CLAUDE.md がある場合は Multi-Claude 設定を追加。

# 元の CLAUDE.md 内容

# 🤖 Multi-Claude システム設定

## Agent Communication System

### エージェント構成

- **PRESIDENT** (別セッション): 統括責任者 + 動的指示書生成
- **boss1** (multiagent:0.0): チームリーダー + 指示書読み込み
- **worker1,2,3** (multiagent:0.1-3): 実行担当 + 指示書実行

### あなたの役割（動的版）

- **PRESIDENT**: @instructions/president_dynamic.md
- **boss1**: @instructions/boss_dynamic.md
- **worker1,2,3**: @instructions/worker_dynamic.md

### メッセージ送信

```bash
./agent-send.sh [相手] "[メッセージ]"
```

### 新しい基本フロー

ユーザー → PRESIDENT（指示書生成） → boss1（指示書読み込み） → workers（指示書実行） → boss1 → PRESIDENT

### 動的指示書システム

- ユーザーの要求に応じて PRESIDENT が指示書を自動生成
- BOSS と WORKER は生成された指示書を読み込んで実行
- 柔軟でスケーラブルなタスク管理が可能

## 🚀 CI/CD システム

### 自動リリースパイプライン

#### リリースフロー

```
1. git tag v1.0.X -m "Release message"
2. git push origin v1.0.X
3. GitHub Actions 自動実行
4. Homebrew Formula 自動更新
```

#### GitHub Actions ワークフロー

##### 1. update-homebrew.yml (自動実行)

- **トリガー**: タグプッシュ (v\*)
- **処理**:
  1. tarball URL 生成
  2. SHA256 計算
  3. homebrew-multi-claude リポジトリへ自動更新
  4. 約 15-18 秒で完了

##### 2. update-homebrew-manual.yml (手動実行)

- **用途**: 特定バージョンの再配布
- **実行**: GitHub Actions 画面から手動トリガー

### CI/CD 設定要件

#### 必要なシークレット

- `HOMEBREW_GITHUB_TOKEN`: homebrew-multi-claude リポジトリへのアクセス権限
  - 設定場所: Settings > Secrets and variables > Actions
  - 必要権限: `repo` (Full control)

### リポジトリ構成

```
multi-claude/    # 開発リポジトリ
├── .github/workflows/       # CI/CD設定
├── multi-claude            # 実行ファイル
├── instructions/           # 動的指示書
└── CLAUDE.md              # プロジェクト設定

homebrew-multi-claude/      # 配布リポジトリ
├── Formula/               # Homebrew Formula
└── README.md             # インストール手順
```

### 開発ワークフロー

1. **機能開発**: feature/\* ブランチで開発
2. **テスト**: ローカルで動作確認
3. **マージ**: main ブランチへマージ
4. **リリース**: タグ付けで自動配布

### バージョニング規則

- **v1.0.X**: パッチリリース（バグ修正）
- **v1.X.0**: マイナーリリース（機能追加）
- **vX.0.0**: メジャーリリース（破壊的変更）
