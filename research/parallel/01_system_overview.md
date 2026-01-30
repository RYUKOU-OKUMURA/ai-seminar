# Claude Code 並列実行システム概要

> **Version**: 1.0.0
> **Created**: 2026-01-29
> **Author**: Ashigaru1 (with Senior Software Engineer persona)

---

## 1. 並列実行システムとは

### 1.1 基本概念

Claude Codeの並列実行システムは、**複数のAIエージェントを同時に起動し、それぞれが独立したコンテキストで作業を行う仕組み**です。

```
従来の逐次実行          並列実行
─────────────────     ─────────────────
Task1 → Task2 → Task3  Task1 ─┐
                         Task2 ├→ 同時に実行
                         Task3 ┘
```

**メリット**
- 作業時間の短縮（理論上、1/Nに圧縮可能）
- コンテキストの分離（各エージェントが独立して思考可能）
- 専門化（役割分担による品質向上）

---

## 2. システムアーキテクチャ

### 2.1 全体構成図

```mermaid
graph TB
    subgraph "Human Layer"
        User[上様/ユーザー]
    end

    subgraph "Orchestration Layer"
        Shogun[将軍 Shogun<br/>プロジェクト統括]
        Karo[家老 Karo<br/>タスク管理・分配]
    end

    subgraph "Execution Layer"
        A1[足軽1<br/>Ashigaru1]
        A2[足軽2<br/>Ashigaru2]
        A3[足軽3<br/>Ashigaru3]
        AN[足軽N<br/>...]
    end

    subgraph "Communication Layer"
        YAML[YAMLファイル<br/>指示・報告]
        TMUX[tmux send-keys<br/>通知]
    end

    User -->|指示| Shogun
    Shogun -->|YAML| Karo
    Karo -->|YAML| A1
    Karo -->|YAML| A2
    Karo -->|YAML| A3
    Karo -->|YAML| AN
    A1 -->|報告| Karo
    A2 -->|報告| Karo
    A3 -->|報告| Karo
    AN -->|報告| Karo

    Karo -.->|通知| TMUX
    TMUX -.->|wake-up| A1
    TMUX -.->|wake-up| A2

    style User fill:#f9f,stroke:#333,stroke-width:2px
    style Shogun fill:#ff9,stroke:#333,stroke-width:2px
    style Karo fill:#9f9,stroke:#333,stroke-width:2px
    style A1 fill:#99f,stroke:#333,stroke-width:2px
    style A2 fill:#99f,stroke:#333,stroke-width:2px
    style A3 fill:#99f,stroke:#333,stroke-width:2px
```

### 2.2 通信プロトコル

```mermaid
sequenceDiagram
    participant User as 上様
    participant Shogun as 将軍
    participant Karo as 家老
    participant A1 as 足軽1
    participant A2 as 足軽2

    User->>Shogun: プロジェクト指示
    Shogun->>Karo: shogun_to_karo.yaml
    Karo->>A1: ashigaru1.yaml
    Karo->>A2: ashigaru2.yaml

    Note over A1,A2: 並列で作業実行

    A1->>Karo: ashigaru1_report.yaml
    A2->>Karo: ashigaru2_report.yaml

    Karo->>Shogun: 進捗集約
    Shogun->>User: dashboard.md 更新
```

**重要な設計思想**
- **イベント駆動**: ポーリング禁止（API代金節約）
- **非同期通信**: 各エージェントが自分のペースで作業
- **YAMLベース**: 指示・報告を構造化されたファイルでやり取り

---

## 3. 並列実行のパターン

### 3.1 パターン分類

| パターン | 説明 | 適用例 |
|----------|------|--------|
| **分業型** | タスクを分割して各エージェントが担当 | ドキュメントの章別執筆 |
| **投票型** | 同じタスクを複数エージェントで実行し結果を比較 | コードレビーの多重チェック |
| **パイプライン型** | 出力を次のエージェントの入力にする | 翻訳→校正→フォーマット |
| **競争型** | 最初に完了した結果を採用 | 複数アプローチの並列探索 |

### 3.2 分業型の詳細

```mermaid
graph LR
    subgraph "元のタスク"
        BIG[大規模なレポート作成]
    end

    subgraph "タスク分割"
        T1[第1章: はじめに]
        T2[第2章: 技術解説]
        T3[第3章: 事例紹介]
        T4[第4章: まとめ]
    end

    subgraph "並列実行"
        A1[足軽1]
        A2[足軽2]
        A3[足軽3]
        A4[足軽4]
    end

    subgraph "結果統合"
        FINAL[完成したレポート]
    end

    BIG --> T1
    BIG --> T2
    BIG --> T3
    BIG --> T4

    T1 --> A1
    T2 --> A2
    T3 --> A3
    T4 --> A4

    A1 --> FINAL
    A2 --> FINAL
    A3 --> FINAL
    A4 --> FINAL
```

### 3.3 パイプライン型の詳細

```mermaid
graph LR
    A[原文] --> B[翻訳エージェント]
    B --> C[訳文]
    C --> D[校正エージェント]
    D --> E[校正済み]
    E --> F[フォーマットエージェント]
    F --> G[完成版]
```

---

## 4. Claude Codeでの実装方式

### 4.1 Task Toolによるサブエージェント起動

Claude Codeには `Task` ツールがあり、これを使ってサブエージェントを起動できます。

```mermaid
graph TB
    MAIN[メインエージェント<br/>Claude Code本体] -->|Task Tool| SA1[サブエージェント1<br/>general-purpose]
    MAIN -->|Task Tool| SA2[サブエージェント2<br/>general-purpose]
    MAIN -->|Task Tool| SA3[サブエージェント3<br/>Explore]

    SA1 -->|結果| MAIN
    SA2 -->|結果| MAIN
    SA3 -->|結果| MAIN

    MAIN -->|統合| RESULT[最終成果物]
```

**実装例（擬似コード）**

```python
# Claude Codeでの並列実行イメージ
results = await asyncio.gather(
    Task("general-purpose", "章1を執筆"),
    Task("general-purpose", "章2を執筆"),
    Task("general-purpose", "章3を執筆"),
)

# 結果を統合
final_document = merge(results)
```

### 4.2 tmux + 複数セッション方式

multi-agent-shogunで採用している方式です。

```mermaid
graph TB
    subgraph "tmuxセッション: shogun"
        P0[Pane 0: 将軍]
    end

    subgraph "tmuxセッション: multiagent"
        P1[Pane 0: 家老]
        P2[Pane 1: 足軽1]
        P3[Pane 2: 足軽2]
        P4[Pane 3: 足軽3]
    end

    P0 -.->|send-keys| P1
    P1 -.->|send-keys| P2
    P1 -.->|send-keys| P3
    P1 -.->|send-keys| P4

    style P0 fill:#ff9,stroke:#333
    style P1 fill:#9f9,stroke:#333
    style P2 fill:#99f,stroke:#333
    style P3 fill:#99f,stroke:#333
    style P4 fill:#99f,stroke:#333
```

**メリット**
- 各ペインで独立したClaude Codeセッションが動作
- 人間がリアルタイムで状況を監視可能
- YAMLファイルによる永続的な指示・報告の記録

---

## 5. コンテキスト管理の重要性

### 5.1 コンテキストウィンドウの問題

```mermaid
graph TB
    subgraph "単一エージェント"
        S1[タスク1] --> S2[タスク2]
        S2 --> S3[タスク3]
        S3 --> S4[タスク4]
        CTX[コンテキスト蓄積<br/>↓空き容量]
        CTX -.->|20%以下| WARNING[⚠️/compact実行必要]
    end

    subgraph "並列エージェント"
        P1[エージェント1<br/>独立コンテキスト]
        P2[エージェント2<br/>独立コンテキスト]
        P3[エージェント3<br/>独立コンテキスト]
        P1 -->|結果のみ統合| MAIN_CTX
        P2 -->|結果のみ統合| MAIN_CTX
        P3 -->|結果のみ統合| MAIN_CTX
    end
```

### 5.2 コンテキスト分散のベストプラクティス

| 対策 | 説明 | 効果 |
|------|------|------|
| Task Tool活用 | サブエージェントに作業を委譲 | メインコンテキストを肥大化させない |
| /compact実行 | 古い会話を要約 | 空き容量を確保 |
| 結果のみ統合 | サブエージェントの詳細ログは破棄 | 必要な情報だけを保持 |

---

## 6. 用語集

| 用語 | 読み | 説明 |
|------|------|------|
| 将軍 | しょうぐん | プロジェクト全体を統括するエージェント |
| 家老 | かろう | タスクの管理・分配を行うエージェント |
| 足軽 | あしがる | 実際の作業を行う実働エージェント |
| 上様 | うえざま | ユーザー（人間）の敬称 |
| ポーリング | polling | 定期的に状態を確認しに行くこと（非推奨） |
| イベント駆動 | event-driven | 状態変化時にのみ通知を受け取る方式 |

---

## 7. まとめ

Claude Codeの並列実行システムは、以下の要素で構成されます：

1. **階層構造**: 将軍 → 家老 → 足軽
2. **通信プロトコル**: YAMLファイル + tmux send-keys
3. **並列パターン**: 分業型・投票型・パイプライン型・競争型
4. **実装方式**: Task Tool または tmux複数セッション
5. **コンテキスト管理**: 分散と要約で効率化

次章では、具体的な実装手順について詳しく解説します。
