# Claude Code並列実行の環境構築ガイド

## 1. 概要

### 1.1 システムの全体像

Claude Code並列実行環境は、**tmux**（ターミナルマルチプレクサ）と**Claude Code CLI**を組み合わせたマルチエージェントシステムです。1つのコンピュータで複数のAIエージェントを同時に起動し、それぞれが独立したタスクを並行処理できる環境を構築します。

```
┌─────────────────────────────────────────────────────────────────┐
│                        あなたのコンピュータ                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────┐         ┌──────────────────────────┐      │
│  │  tmuxセッション1  │         │  tmuxセッション2         │      │
│  │  (shogun)        │         │  (multiagent)            │      │
│  │                  │         │                          │      │
│  │  ┌────────────┐  │         │  ┌─────┬─────┬─────┐    │      │
│  │  │   将軍     │  │         │  │家老 │足軽1│足軽2│    │      │
│  │  │(Shogun)    │  │         │  │(Karo)│     │     │    │      │
│  │  │            │  │         │  ├─────┼─────┼─────┤    │      │
│  │  │ 総括・司令  │  │         │  │足軽3│足軽4│足軽5│    │      │
│  │  └────────────┘  │         │  │     │     │     │    │      │
│  │                  │         │  ├─────┼─────┼─────┤    │      │
│  └──────────────────┘         │  │足軽6│足軽7│足軽8│    │      │
│                                │  │     │     │     │    │      │
│                                │  └─────┴─────┴─────┘    │      │
│                                └──────────────────────────┘      │
│                                         ▲                       │
│                                         │ YAMLファイル           │
│                                         │ send-keysで通信       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 構成要素の説明

| 構成要素 | 役割 | 説明 |
|---------|------|------|
| **tmux** | ターミナル分割 | 1つのターミナル画面を複数のペイン（分割画面）に分けるソフトウェア |
| **Claude Code CLI** | AIエージェント | Anthropic社のClaudeをコマンドラインから利用できるツール |
| **YAMLファイル** | 通信手段 | エージェント間の指示・報告を記録するテキストファイル |
| **send-keysコマンド** | 通知方法 | tmuxを使って、あるペインから別のペインにキー入力を送るコマンド |
| **ペイン（Pane）** | 作業領域 | tmuxで分割された1つ1つの画面。それぞれで独立したClaude Codeが動作 |

### 1.3 なぜ並列実行が必要か

- **時間短縮**: 複数のタスクを同時に実行できる
- **効率化**: 待ち時間が発生しても、他のエージェントが作業を継続
- **専門化**: 各エージェントが特定の役割（開発、テスト、ドキュメント等）を担当

## 2. 前提条件

### 2.1 必要なソフトウェア

| ソフトウェア | 最小バージョン | 用途 | 確認コマンド |
|-------------|---------------|------|-------------|
| **tmux** | 3.0以上 | ターミナル分割 | `tmux -V` |
| **Node.js** | 18.0以上 | Claude Code実行環境 | `node -v` |
| **npm** | 9.0以上 | パッケージ管理 | `npm -v` |
| **Claude Code CLI** | 最新版 | AIエージェント本体 | `claude --version` |

### 2.2 必要なアカウント等

- **Anthropicアカウント**: Claude APIを使用するためのアカウント
- **APIキー**: AnthropicのAPIキー（`ANTHROPIC_API_KEY`環境変数に設定）

### 2.3 推奨環境

- **OS**: macOS, Linux, WSL2（Windows Subsystem for Linux）
- **メモリ**: 8GB以上（16GB推奨）
- **CPU**: 複数コア推奨（並列処理のため）

## 3. tmuxのインストールと基本操作

### 3.1 インストール方法

#### macOSの場合

```bash
# Homebrewを使用している場合
brew install tmux
```

#### Linux（Ubuntu/Debian）の場合

```bash
# aptパッケージマネージャを使用
sudo apt-get update
sudo apt-get install tmux
```

#### WSL2（Windows Subsystem for Linux）の場合

```bash
# Ubuntu on WSL2の場合
sudo apt-get update
sudo apt-get install tmux
```

#### インストール確認

```bash
# バージョンを確認
tmux -V
# 出力例: tmux 3.3a
```

### 3.2 tmuxの基本操作

#### 用語の説明

| 用語 | 説明 | 図解 |
|------|------|------|
| **セッション（Session）** | tmuxのインスタンス。1つのセッションで複数のウィンドウを持てる | `セッション ──┬── ウィンドウ1`<br>`             └── ウィンドウ2` |
| **ウィンドウ（Window）** | セッション内のタブ。1つのウィンドウで複数のペインを持てる | `ウィンドウ ──┬── ペイン1`<br>`             └── ペイン2` |
| **ペイン（Pane）** | ウィンドウを分割した1つ1つの画面 | `ペイン: 実際にコマンドを入力する場所` |

#### プレフィックスキーについて

tmuxのコマンドは**プレフィックスキー**を先に押してから入力します。

- **デフォルト**: `Ctrl-b`（Ctrlを押しながらbを押す）
- **本ガイドでの推奨**: `Ctrl-b`（デフォルトのまま）

```
例: ペインを縦に分割する場合
1. Ctrl-b を押す（プレフィックス）
2. % を押す（分割コマンド）
```

#### セッションの作成

```bash
# 新しいセッションを作成して接続
tmux new-session -s mysession

# セッション名を省略すると数字が割り振られる
tmux new-session
```

**期待される結果**:
- 新しいウィンドウが開き、プロンプトが表示される
- ウィンドウ下部にステータスバーが表示される

#### セッションのデタッチ（切り離し）

```bash
# セッションから切り離す（セッションはバックグラウンドで継続）
# 方法1: プレフィックスキー + d
Ctrl-b, d

# 方法2: コマンド入力
tmux detach-client
```

**期待される結果**:
- tmuxセッションから抜け、通常のターミナルに戻る
- セッション内のプロセスは継続実行中

#### セッションのアタッチ（再接続）

```bash
# セッション一覧を表示
tmux list-sessions
# 出力例:
# mysession: 1 windows (created Tue Jan 28 10:00:00 2026)

# セッションに再接続
tmux attach-session -t mysession
# 短縮形:
tmux attach -t mysession
# さらに短縮（セッション名を省略すると最新のセッションに接続）:
tmux a
```

**期待される結果**:
- 以前のセッションが復元され、続きから作業できる

#### セッションの終了

```bash
# 方法1: セッション内でexitコマンド
exit

# 方法2: セッション外から削除
tmux kill-session -t mysession

# 全セッションを削除
tmux kill-server
```

### 3.3 ペイン分割

#### 基本の分割操作

| 操作 | コマンド | 説明 |
|------|---------|------|
| 縦に分割 | `Ctrl-b, %` | 左右に2分割 |
| 横に分割 | `Ctrl-b, "` | 上下に2分割 |
| ペインを移動 | `Ctrl-b, 方向キー` または `Ctrl-b, o` | 次のペインへ移動 |
| ペインを閉じる | `exit` または `Ctrl-d` | 現在のペインを終了 |
| ペインを入れ替え | `Ctrl-b, Ctrl-o` | ペインの配置を回転 |

#### 3x3グリッドの作成例

```bash
# 1. 新しいセッションを作成
tmux new-session -s multiagent

# 2. 最初に縦に2分割
Ctrl-b, %

# 3. さらに縦に分割（合計3列）
Ctrl-b, %

# 4. 左の列を横に3分割にするため、左の列を選択
Ctrl-b, 方向キー（左の列へ移動）

# 5. 横に分割
Ctrl-b, "

# 6. もう一度横に分割（左の列が3行になる）
Ctrl-b, "

# 7. 同様に中央の列、右の列も分割
```

**コマンドラインからの作成**（スクリプト化する場合）:

```bash
# セッション作成
tmux new-session -d -s multiagent

# 3列に分割
tmux split-window -h -t multiagent:0
tmux split-window -h -t multiagent:0

# 左の列を3行に分割
tmux select-pane -t multiagent:0.0
tmux split-window -v
tmux split-window -v

# （同様に他の列も分割）
```

#### ペインの情報表示

```bash
# 全ペインの一覧を表示
tmux list-panes -t multiagent:0
# 出力例:
# 0: [80x24] [history 0/2000, 0 bytes] %0
# 1: [80x24] [history 0/2000, 0 bytes] %1
# ...

# 現在のペイン番号を確認
tmux display-message -p '#P'
# 出力例: 0
```

### 3.4 ペイン間の通信

#### send-keysコマンドの基本

```bash
# 基本構文
tmux send-keys -t {セッション名}:{ウィンドウ番号}.{ペイン番号} 'コマンド' Enter

# 具体例
tmux send-keys -t multiagent:0.0 'echo Hello' Enter
```

**構成要素の説明**:
- `multiagent`: セッション名
- `0`: ウィンドウ番号（0から始まる）
- `0`: ペイン番号（0から始まる）
- `'echo Hello'`: 送信するテキスト
- `Enter`: エンターキーを送信

#### 重要: 2回に分けて送信

```bash
# ❌ 誤った方法（1行で記述）
tmux send-keys -t multiagent:0.0 'echo Hello' Enter
# → Enterが正しく解釈されない場合がある

# ✅ 正しい方法（2回に分ける）
# 1回目: テキストを送信
tmux send-keys -t multiagent:0.0 'echo Hello'

# 2回目: Enterを送信
tmux send-keys -t multiagent:0.0 Enter
```

**なぜ2回に分けるのか**:
- 1回のBash呼び出しでEnterキーが正しく解釈されない場合がある
- 2回に分けることで、確実にコマンドを実行できる

#### ペインの内容を確認

```bash
# ペインの内容（テキスト）を取得
tmux capture-pane -t multiagent:0.0 -p

# 最新の20行のみを取得
tmux capture-pane -t multiagent:0.0 -p | tail -20
```

**使用例**: エージェントが作業中か確認する場合
```bash
# 足軽1が作業中か確認
tmux capture-pane -t multiagent:0.1 -p | tail -20
# "thinking"や"Esc to interrupt"が表示されていれば作業中
```

## 4. Claude Codeの設定

### 4.1 インストール

#### Node.jsとnpmの確認

```bash
# Node.jsのバージョン確認
node -v
# 推奨: v18.0.0以上

# npmのバージョン確認
npm -v
# 推奨: v9.0.0以上
```

#### Node.jsのインストール（未インストールの場合）

**nvmを使用する方法（推奨）**:

```bash
# nvm（Node Version Manager）をインストール
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash

# ターミナルを再起動後、Node.jsをインストール
nvm install 20
nvm use 20

# デフォルトバージョンとして設定
nvm alias default 20
```

**直接インストールする方法（Ubuntu/Debian）**:

```bash
# NodeSourceリポジトリを追加
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# Node.jsをインストール
sudo apt-get install -y nodejs
```

#### Claude Code CLIのインストール

```bash
# グローバルにインストール
npm install -g @anthropic-ai/claude-code

# インストール確認
claude --version
```

**期待される結果**:
- バージョン情報が表示される
- `claude`コマンドがどこからでも実行可能

#### APIキーの設定

```bash
# Anthropic APIキーを環境変数に設定
export ANTHROPIC_API_KEY='sk-ant-xxxxx'

# 永続的に設定する場合（~/.bashrc または ~/.zshrc に追加）
echo 'export ANTHROPIC_API_KEY="sk-ant-xxxxx"' >> ~/.bashrc
source ~/.bashrc
```

### 4.2 設定ファイル

#### Claude Codeの設定場所

| OS | 設定ファイルの場所 |
|----|-------------------|
| macOS/Linux | `~/.claude/` |
| WSL2 | `~/.claude/` |

#### 設定ファイルの構成

```
~/.claude/
├── settings.json           # Claude Code全体の設定
├── agents/                 # エージェント（スキル）の保存先
└── skills/                 # カスタムスキルの保存先
```

#### settings.jsonの例

```json
{
  "model": "claude-sonnet-4-5-20251101",
  "maxTokens": 8192,
  "temperature": 0.7,
  "autoApproveTools": false,
  "allowedTools": [
    "Bash",
    "Read",
    "Write",
    "Edit",
    "Glob",
    "Grep"
  ]
}
```

**主要設定項目の説明**:

| 項目 | 説明 | 推奨値 |
|------|------|--------|
| `model` | 使用するClaudeのモデル | `claude-sonnet-4-5-20251101` |
| `maxTokens` | 最大トークン数 | `8192` |
| `temperature` | 応答のランダム性（0-1） | `0.7` |
| `autoApproveTools` | ツール使用の自動承認 | `false`（セキュリティのため） |
| `allowedTools` | 許可するツールの一覧 | 必要なツールのみ指定 |

## 5. 並列実行環境の構築

### 5.1 セッションの作成

#### 最小構成のセッション構成

最小構成として、以下のセッションを作成します：

- **shogunセッション**: 将軍1名（1ペイン）
- **multiagentセッション**: 家老1名、足軽2名（合計3ペイン）

#### セッション作成スクリプト

```bash
#!/bin/bash
# setup_minimal.sh - 最小構成のセットアップスクリプト

# 既存セッションの削除
tmux kill-session -t shogun 2>/dev/null
tmux kill-session -t multiagent 2>/dev/null

# multiagentセッションの作成（3ペイン: 家老 + 足軽2名）
tmux new-session -d -s multiagent
tmux split-window -h -t multiagent:0
tmux select-pane -t multiagent:0.0
tmux split-window -v

# ペインタイトルの設定
tmux select-pane -t multiagent:0.0 -T "karo"
tmux select-pane -t multiagent:0.1 -T "ashigaru1"
tmux select-pane -t multiagent:0.2 -T "ashigaru2"

# shogunセッションの作成（1ペイン）
tmux new-session -d -s shogun

echo "セッションの作成完了"
tmux list-sessions
```

**期待される結果**:
```
multiagent: 3 windows (created ...)
shogun: 1 windows (created ...)
```

### 5.2 ペインの分割

#### multiagentセッションのレイアウト

```
┌─────────────────────────────────┐
│  multiagentセッション (3ペイン) │
├──────────────┬──────────────────┤
│   karo       │   ashigaru1      │
│   (家老)     │   (足軽1)        │
│              │                  │
├──────────────┴──────────────────┤
│           ashigaru2             │
│           (足軽2)                │
│                                  │
└─────────────────────────────────┘
```

#### コマンドラインからのペイン分割

```bash
# multiagentセッションの作成
tmux new-session -d -s multiagent

# 縦に分割（左右2分割）
tmux split-window -h -t multiagent:0

# 左のペイン(0.0)を選択して横に分割
tmux select-pane -t multiagent:0.0
tmux split-window -v

# これで3つのペインができる:
# - 0.0: karo（左上）
# - 0.1: ashigaru1（右上）
# - 0.2: ashigaru2（下半分）
```

#### ペイン番号の確認

```bash
# 全ペインの一覧
tmux list-panes -t multiagent:0 -F "#{pane_index}: #{pane_title}"
```

**期待される出力**:
```
0: karo
1: ashigaru1
2: ashigaru2
```

### 5.3 各ペインでClaude Codeを起動

#### 手動で起動する方法

```bash
# 1. shogunセッションにアタッチ
tmux attach -t shogun

# 2. Claude Codeを起動（モデル指定）
claude --model opus

# 3. デタッチ
Ctrl-b, d

# 4. multiagentセッションにアタッチ
tmux attach -t multiagent

# 5. 各ペインでClaude Codeを起動
# karo（ペイン0）で:
claude --model sonnet

# ashigaru1（ペイン1）に移動して:
Ctrl-b, o  # 次のペインへ移動
claude --model haiku

# ashigaru2（ペイン2）に移動して:
Ctrl-b, o  # 次のペインへ移動
claude --model haiku
```

#### send-keysで起動する方法（推奨）

```bash
#!/bin/bash
# start_claude.sh - 全ペインでClaude Codeを起動

# shogun（将軍）
tmux send-keys -t shogun 'claude --model opus'
tmux send-keys -t shogun Enter

# karo（家老）
tmux send-keys -t multiagent:0.0 'claude --model sonnet'
tmux send-keys -t multiagent:0.0 Enter

# ashigaru1（足軽1）
tmux send-keys -t multiagent:0.1 'claude --model haiku'
tmux send-keys -t multiagent:0.1 Enter

# ashigaru2（足軽2）
tmux send-keys -t multiagent:0.2 'claude --model haiku'
tmux send-keys -t multiagent:0.2 Enter

echo "Claude Codeの起動完了"
```

**期待される結果**:
- 各ペインでClaude Codeが起動し、プロンプトが表示される
- 各ペインで独立してClaudeと対話できる状態になる

#### モデル選択の目安

| エージェント | 推奨モデル | 理由 |
|-------------|-----------|------|
| 将軍（Shogun） | Opus | 高度な戦略立案・統括が必要 |
| 家老（Karo） | Sonnet | タスク管理・配分のバランスが良い |
| 足軽（Ashigaru） | Haiku | 軽量作業・コスト重視 |

## 6. 通信プロトコルの設定

### 6.1 YAMLファイルによる指示・報告

#### ディレクトリ構造

```
multi-agent-shogun/
├── queue/
│   ├── shogun_to_karo.yaml        # 将軍→家老の指示キュー
│   ├── karo_to_ashigaru.yaml      # 家老→足軽の割当テーブル
│   ├── tasks/                     # 各足軽専用タスクファイル
│   │   ├── ashigaru1.yaml
│   │   ├── ashigaru2.yaml
│   │   └── ...
│   └── reports/                   # 各足軽専用報告ファイル
│       ├── ashigaru1_report.yaml
│       ├── ashigaru2_report.yaml
│       └── ...
├── config/
│   ├── projects.yaml              # プロジェクト一覧
│   └── settings.yaml              # システム設定
└── dashboard.md                   # 人間用ダッシュボード
```

#### YAMLファイルの例

**将軍→家老の指示（shogun_to_karo.yaml）**:

```yaml
queue:
  - id: cmd_001
    timestamp: "2026-01-29T10:00:00"
    command: "調査資料を作成せよ"
    project: ai_seminar
    priority: high
    status: pending
    context: |
      Claude Codeの並列実行環境について調査し、
      Markdownで資料を作成せよ。
```

**家老→足軽の割当（ashigaru1.yaml）**:

```yaml
task:
  task_id: subtask_001
  parent_cmd: cmd_001
  description: "環境構築手順をまとめよ"
  target_path: "/path/to/output.md"
  status: assigned
  timestamp: "2026-01-29T10:05:00"
```

**足軽→家老の報告（ashigaru1_report.yaml）**:

```yaml
worker_id: ashigaru1
task_id: subtask_001
timestamp: "2026-01-29T10:30:00"
status: done
result:
  summary: "環境構築手順の作成完了"
  files_modified:
    - "/path/to/output.md"
  notes: "全ステップ完了"
```

### 6.2 tmux send-keys による通知

#### 通知の基本フロー

```
1. 指示書く（YAMLファイル更新）
       ↓
2. send-keysで相手を起こす
       ↓
3. 相手がYAMLを読んで実行
       ↓
4. 完了したら報告書書く（YAMLファイル更新）
       ↓
5. send-keysで上官を起こす
```

#### 将軍→家老の通知

```bash
#!/bin/bash
# 将軍から家老への通知

# 1. YAMLファイルに指示を書く（エディタ等で事前に作成）

# 2. 家老を起こす
tmux send-keys -t multiagent:0.0 'queue/shogun_to_karo.yaml に新しい指示がある。確認して実行せよ。'
tmux send-keys -t multiagent:0.0 Enter
```

#### 家老→足軽の通知

```bash
#!/bin/bash
# 家老から足軽への通知

# 1. YAMLファイルにタスクを書く（ashigaru1.yaml等）

# 2. 足軽を起こす
tmux send-keys -t multiagent:0.1 'queue/tasks/ashigaru1.yaml に任務がある。確認して実行せよ。'
tmux send-keys -t multiagent:0.1 Enter
```

#### 足軽→家老の報告

```bash
#!/bin/bash
# 足軽から家老への報告

# 1. 報告書を書く（ashigaru1_report.yaml）

# 2. 家老を起こす
tmux send-keys -t multiagent:0.0 'ashigaru1、任務完了でござる。報告書を確認されよ。'
tmux send-keys -t multiagent:0.0 Enter
```

### 6.3 イベント駆動通信の実装

#### ポーリング禁止の理由

```
❌ ポーリング方式（やってはいけない）
足軽: 「何かタスクありますか？」（10秒ごとに確認）
足軽: 「何かタスクありますか？」
足軽: 「何かタスクがありますか？」
→ API代金がかさむ

✅ イベント駆動方式（正しい）
家老: send-keysで足軽を起こす
足軽: 起こされた時にだけYAMLを確認
→ API代金節約
```

#### イベント駆動の実装パターン

**パターン1: 単純な通知**

```bash
# YAMLファイル更新後にsend-keys
echo "新しい指示をYAMLに書き込む" > queue/shogun_to_karo.yaml
tmux send-keys -t multiagent:0.0 '指示を確認されよ。'
tmux send-keys -t multiagent:0.0 Enter
```

**パターン2: 確認応答付き**

```bash
# 指示を出す
echo "新しい指示" > queue/shogun_to_karo.yaml
tmux send-keys -t multiagent:0.0 '指示がある。確認後に「了解」と返されよ。'
tmux send-keys -t multiagent:0.0 Enter

# ここで処理を終了し、家老からの応答を待つ
```

**パターン3: ステータス確認**

```bash
# 家老の状態を確認してから指示
STATUS=$(tmux capture-pane -t multiagent:0.0 -p | tail -5)
if [[ $STATUS == *"thinking"* ]]; then
    echo "家老は作業中です"
else
    # 家老を起こす
    tmux send-keys -t multiagent:0.0 '新しい指示がある'
    tmux send-keys -t multiagent:0.0 Enter
fi
```

## 7. 実践例

### 7.1 最小構成の作成（将軍1、家老1、足軽2）

#### 完全なセットアップスクリプト

```bash
#!/bin/bash
# minimal_setup.sh - 最小構成の完全セットアップ

set -e

# 色の定義
GREEN='\033[0;32m'
BLUE='\033[0;34m'
NC='\033[0m'

echo -e "${BLUE}=== Claude Code並列実行環境のセットアップ ===${NC}"

# ============================================================
# STEP 1: ディレクトリ構造の作成
# ============================================================
echo -e "${GREEN}[1/6] ディレクトリ構造を作成中...${NC}"
mkdir -p queue/tasks
mkdir -p queue/reports
mkdir -p config

# ============================================================
# STEP 2: 設定ファイルの初期化
# ============================================================
echo -e "${GREEN}[2/6] 設定ファイルを初期化中...${NC}"

# settings.yaml
cat > config/settings.yaml << 'EOF'
language: ja

skill:
  save_path: "~/.claude/skills/"
  local_path: "./skills/"
EOF

# projects.yaml
cat > config/projects.yaml << 'EOF'
projects:
  - id: sample_project
    name: "Sample Project"
    path: "/path/to/project"
    priority: high
    status: active

current_project: sample_project
EOF

# ============================================================
# STEP 3: キューファイルの初期化
# ============================================================
echo -e "${GREEN}[3/6] キューファイルを初期化中...${NC}"

# shogun_to_karo.yaml
cat > queue/shogun_to_karo.yaml << 'EOF'
queue: []
EOF

# karo_to_ashigaru.yaml
cat > queue/karo_to_ashigaru.yaml << 'EOF'
assignments:
  ashigaru1:
    task_id: null
    description: null
    target_path: null
    status: idle
  ashigaru2:
    task_id: null
    description: null
    target_path: null
    status: idle
EOF

# 足軽用タスクファイル
for i in 1 2; do
    cat > queue/tasks/ashigaru${i}.yaml << EOF
task:
  task_id: null
  parent_cmd: null
  description: null
  target_path: null
  status: idle
  timestamp: ""
EOF
done

# 足軽用レポートファイル
for i in 1 2; do
    cat > queue/reports/ashigaru${i}_report.yaml << EOF
worker_id: ashigaru${i}
task_id: null
timestamp: ""
status: idle
result: null
EOF
done

# ============================================================
# STEP 4: tmuxセッションの作成
# ============================================================
echo -e "${GREEN}[4/6] tmuxセッションを作成中...${NC}"

# 既存セッションの削除
tmux kill-session -t shogun 2>/dev/null || true
tmux kill-session -t multiagent 2>/dev/null || true

# multiagentセッション（3ペイン）
tmux new-session -d -s multiagent
tmux split-window -h -t multiagent:0
tmux select-pane -t multiagent:0.0
tmux split-window -v

# ペインタイトル設定
tmux select-pane -t multiagent:0.0 -T "karo"
tmux select-pane -t multiagent:0.1 -T "ashigaru1"
tmux select-pane -t multiagent:0.2 -T "ashigaru2"

# shogunセッション（1ペイン）
tmux new-session -d -s shogun

# ============================================================
# STEP 5: Claude Codeの起動
# ============================================================
echo -e "${GREEN}[5/6] Claude Codeを起動中...${NC}"

# 将軍（Opus）
tmux send-keys -t shogun 'claude --model opus'
tmux send-keys -t shogun Enter

# 家老（Sonnet）
tmux send-keys -t multiagent:0.0 'claude --model sonnet'
tmux send-keys -t multiagent:0.0 Enter

# 足軽1（Haiku）
tmux send-keys -t multiagent:0.1 'claude --model haiku'
tmux send-keys -t multiagent:0.1 Enter

# 足軽2（Haiku）
tmux send-keys -t multiagent:0.2 'claude --model haiku'
tmux send-keys -t multiagent:0.2 Enter

# ============================================================
# STEP 6: 完了
# ============================================================
echo -e "${GREEN}[6/6] セットアップ完了！${NC}"
echo ""
echo "セッション一覧:"
tmux list-sessions
echo ""
echo "各セッションにアタッチするには:"
echo "  将軍:     tmux attach -t shogun"
echo "  家老・足軽: tmux attach -t multiagent"
echo ""
echo "ペインの移動（multiagentセッション内）:"
echo "  Ctrl-b, o  : 次のペインへ移動"
echo "  Ctrl-b, 方向キー : 指定方向のペインへ移動"
```

#### スクリプトの実行

```bash
# 実行権限を付与
chmod +x minimal_setup.sh

# 実行
./minimal_setup.sh
```

### 7.2 動作確認

#### 動作確認の手順

**1. 将軍（Shogun）にアタッチ**

```bash
tmux attach -t shogun
```

**期待される結果**:
- Claude Codeのプロンプトが表示されている
- モデル名に「opus」が含まれている

**2. 家老・足軽の確認**

```bash
# 将軍セッションからデタッチ
Ctrl-b, d

# multiagentセッションにアタッチ
tmux attach -t multiagent
```

**期待される結果**:
- 3つのペインが分割表示されている
- 各ペインでClaude Codeのプロンプトが表示されている
- 左上が家老（karo）、右上が足軽1（ashigaru1）、下が足軽2（ashigaru2）

**3. 通信テスト**

```bash
# デタッチ
Ctrl-b, d

# 将軍から家老へのテスト通知
tmux send-keys -t shogun '動作確認テストを実行せよ。'
tmux send-keys -t shogun Enter

# 家老を起こす
tmux send-keys -t multiagent:0.0 '将軍からの指示がある。確認されよ。'
tmux send-keys -t multiagent:0.0 Enter
```

**期待される結果**:
- 家老のペインに「将軍からの指示がある...」が入力される
- 家老がshogun_to_karo.yamlを確認する
- 必要に応じて足軽に指示が出される

## 8. トラブルシューティング

### 8.1 よくある問題

| 問題 | 原因 | 解決方法 |
|------|------|----------|
| tmuxコマンドが見つからない | tmuxがインストールされていない | パッケージマネージャでtmuxをインストール |
| セッションが作成できない | 同名のセッションが既存 | `tmux kill-session -t セッション名` で削除 |
| send-keysでコマンドが実行されない | Enterが正しく送信されていない | 2回に分けてsend-keysを実行 |
| Claude Codeが起動しない | APIキーが設定されていない | `export ANTHROPIC_API_KEY='sk-ant-...'` を実行 |
| ペインが見つからない | ペイン番号が間違っている | `tmux list-panes` で確認 |
| YAMLファイルが読み込めれない | パスが間違っている | 絶対パスで指定 |

### 8.2 解決方法

#### tmux関連の問題

**問題**: tmuxコマンドが見つからない

```bash
# 解決方法: tmuxをインストール

# macOS
brew install tmux

# Ubuntu/Debian
sudo apt-get install tmux

# 確認
tmux -V
```

**問題**: セッションが既存で作成できない

```bash
# 解決方法: 既存セッションを削除

# セッション一覧を確認
tmux list-sessions

# 特定のセッションを削除
tmux kill-session -t セッション名

# 全セッションを削除
tmux kill-server
```

**問題**: ペイン番号がわからない

```bash
# 解決方法: ペイン一覧を表示

# 全ペインの情報を表示
tmux list-panes -t セッション名

# ペイン番号とタイトルを表示
tmux list-panes -t セッション名 -F "#{pane_index}: #{pane_title}"

# 出力例:
# 0: karo
# 1: ashigaru1
# 2: ashigaru2
```

#### send-keys関連の問題

**問題**: コマンドが実行されない

```bash
# 解決方法: 2回に分けて送信

# ❌ 誤った方法
tmux send-keys -t multiagent:0.0 'echo Hello' Enter

# ✅ 正しい方法
tmux send-keys -t multiagent:0.0 'echo Hello'
tmux send-keys -t multiagent:0.0 Enter
```

**問題**: 相手が反応しない

```bash
# 解決方法: 相手の状態を確認

# ペインの内容を確認
tmux capture-pane -t multiagent:0.0 -p | tail -20

# "thinking"等が表示されていれば作業中
# プロンプト（❯）が表示されていれば待機中
```

#### Claude Code関連の問題

**問題**: Claude Codeが起動しない

```bash
# 解決方法: APIキーを確認

# APIキーが設定されているか確認
echo $ANTHROPIC_API_KEY

# 設定されていない場合は設定
export ANTHROPIC_API_KEY='sk-ant-xxxxx'

# 永続的に設定（~/.bashrc に追加）
echo 'export ANTHROPIC_API_KEY="sk-ant-xxxxx"' >> ~/.bashrc
source ~/.bashrc
```

**問題**: モデルが選択できない

```bash
# 解決方法: 利用可能なモデルを確認

# Claude Codeのヘルプを確認
claude --help

# モデルを明示的に指定
claude --model claude-opus-4-5-20251101
```

#### YAMLファイル関連の問題

**問題**: YAMLファイルが読み込めない

```bash
# 解決方法: パスを絶対パスで指定

# ❌ 相対パス（失敗する場合がある）
./queue/tasks/ashigaru1.yaml

# ✅ 絶対パス
/home/user/multi-agent-shogun/queue/tasks/ashigaru1.yaml

# またはホームディレクトリからのパス
~/multi-agent-shogun/queue/tasks/ashigaru1.yaml
```

**問題**: YAMLの構文エラー

```bash
# 解決方法: YAMLの構文チェック

# yamllintがインストールされている場合
yamllint queue/tasks/ashigaru1.yaml

# インストール
pip install yamllint
# または
apt-get install yamllint
```

## 9. まとめ

### 9.1 本ガイドで学んだこと

1. **tmuxの基本操作**: セッションの作成、ペインの分割、アタッチ/デタッチ
2. **Claude Codeの設定**: インストール、APIキーの設定、モデル選択
3. **通信プロトコル**: YAMLファイルとsend-keysによるイベント駆動通信
4. **環境構築**: 最小構成（将軍1、家老1、足軽2）の完全なセットアップ
5. **トラブルシューティング**: よくある問題とその解決方法

### 9.2 次のステップ

1. **拡張**: 足軽の数を増やす（3名→8名など）
2. **自動化**: セットアップスクリプトの改良
3. **監視**: ダッシュボードの実装
4. **スキル化**: 繰り返しのパターンをスキルとして抽出

### 9.3 参考リソース

- **tmux公式ドキュメント**: https://github.com/tmux/tmux/wiki
- **Claude Code公式**: https://docs.anthropic.com/en/docs/build-with-claude/claude-code
- **YAML公式**: https://yaml.org/

### 9.4 用語集

| 用語 | 説明 |
|------|------|
| tmux | ターミナルマルチプレクサ。1つのターミナルで複数のセッションを管理できるツール |
| セッション | tmuxのインスタンス。複数のウィンドウを持てる |
| ウィンドウ | セッション内のタブ。複数のペインを持てる |
| ペイン | ウィンドウを分割した1つ1つの画面 |
| send-keys | tmuxのコマンド。特定のペインにキー入力を送信できる |
| アタッチ | tmuxセッションに接続すること |
| デタッチ | tmuxセッションから切り離すこと（セッションは継続） |
| イベント駆動 | 何かが起きた時だけ処理を実行する方式（ポーリングの対義語） |
| ポーリング | 定期的に状態を確認する方式（非推奨） |

---

以上で「Claude Code並列実行の環境構築ガイド」を完了とします。本ガイドを参考に、あなたのマルチエージェントシステムを構築してください。ご質問があれば、各セクションのトラブルシューティングを参照してください。
