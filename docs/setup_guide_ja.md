# セットアップガイド（ターミナル2つで「将軍」と「マルチエージェント」を同時表示）

このガイドは、親リポジトリ（例: `ai-seminar/`）配下に `multi-agent-shogun/` を置いて使う構成を前提にしています。

- 作業ディレクトリ: 親リポジトリ（例: `ai-seminar/`）
- システム本体: `ai-seminar/multi-agent-shogun/`
- Claude Code 起動コマンド: `ccz`（`multi-agent-shogun/ccz`）

---

## 0. 前提

- `tmux` が使えること
- Claude Code CLI（`claude`）がインストール済みで、認証/利用できること
- bash/zsh で実行する（Mac/Linux/WSL を想定）

---

## 1. 初回セットアップ（最初の1回だけ）

親リポジトリ直下で実行します。

```bash
cd /path/to/ai-seminar

# 実行権限（初回のみ）
chmod +x multi-agent-shogun/*.sh multi-agent-shogun/ccz

# 依存関係セットアップ（初回のみ）
./multi-agent-shogun/first_setup.sh
```

---

## 2. 毎回の起動（将軍 + マルチエージェントを同時に起動）

```bash
cd /path/to/ai-seminar
./multi-agent-shogun/shutsujin_departure.sh
```

注意:
- 実行すると既存の tmux セッション `shogun` / `multiagent` は一度終了（kill）されます
- キュー/レポートもリセットされます（前回状態を引き継がない前提）

---

## 3. いちばん分かりやすい表示方法（ターミナルを2つ開く）

起動後、ターミナルを2つ開いてそれぞれ attach します。

ターミナルA（将軍）:
```bash
tmux attach-session -t shogun
```

ターミナルB（マルチエージェント）:
```bash
tmux attach-session -t multiagent
```

---

## 4. よく使う操作

バックグラウンドに戻る（処理は継続）:
- `Ctrl+b` → `d`

完全に終了する:
```bash
tmux kill-session -t shogun
tmux kill-session -t multiagent
```

---

## 5. 補足（この構成の動き）

- すべてのエージェントの作業ディレクトリは、親リポジトリ（例: `ai-seminar/`）になります
- Claude Code は `ccz` 経由で起動します（`--dangerously-skip-permissions` をデフォルト付与）
