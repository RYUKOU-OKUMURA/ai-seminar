# Google Apps Script 基本情報とアーキテクチャ

> **Research Date**: 2026-01-29
> **Researcher**: Ashigaru1
> **Task ID**: gas_research_001

---

## 1. Google Apps Script とは

### 1.1 概要

Google Apps Script（GAS）は、Google Workspace 製品との統合とタスクの自動化を実現するクラウドベースの JavaScript プラットフォームである。Google Drive を基盤として動作し、簡潔なコードで Google 製品全体での作業を自動化・拡張することができる。

**公式定義**:
> "Apps Script is a cloud-based JavaScript platform powered by Google Drive that lets you integrate with and automate tasks across Google products."

### 1.2 主な用途

| 用途 | 説明 |
|------|------|
| **Automations（自動化）** | カスタムメニュー、ボタン、ユーザーアクション、時間ベースのスケジュールによって起動されるタスクをプログラムで実行 |
| **Custom Functions（カスタム関数）** | Google Sheets で組み込み関数のように呼び出せる独自の関数を作成 |
| **Add-ons（アドオン）** | Google Workspace 内からサードパーティサービスに接続するアプリを構築し、Google Workspace Marketplace で共有 |
| **Chat apps（チャットアプリ）** | Google Chat ユーザーがサービスと対話できる会話型インターフェースを提供 |
| **AI 機能の統合** | Vertex AI、Gemini などを使用した AI 機能の実装 |

### 1.3 歴史

- **初期**: Mozilla Rhino JavaScript インタプリタを使用（JavaScript 1.6 ベース）
- **現行**: V8 ランタイムを採用（モダンな ECMAScript 構文と機能をサポート）

---

## 2. アーキテクチャ

### 2.1 実行環境

```
┌─────────────────────────────────────────────────────┐
│              Google Cloud Infrastructure            │
├─────────────────────────────────────────────────────┤
│                                                       │
│  ┌───────────────┐      ┌─────────────────────┐   │
│  │   V8 Runtime  │──────│  Apps Script Code   │   │
│  │   (現行)      │      │  (JavaScript)       │   │
│  └───────────────┘      └─────────────────────┘   │
│                                                       │
│  ┌───────────────┐                                   │
│  │ Rhino Runtime │      (レガシー、非推奨)          │
│  │   (旧型)      │                                   │
│  └───────────────┘                                   │
│                                                       │
├─────────────────────────────────────────────────────┤
│           Built-in Services (30+ services)          │
│  GmailApp, SpreadsheetApp, DriveApp, DocumentApp... │
├─────────────────────────────────────────────────────┤
│              Google Workspace Products               │
│  Gmail, Sheets, Docs, Drive, Calendar, Forms...     │
└─────────────────────────────────────────────────────┘
```

### 2.2 サーバーサイド JavaScript

Apps Script は完全にサーバーサイドで実行される：

- **Google のインフラ上で実行**: クライアント側のリソースを消費しない
- **クラウドベース**: Google Drive 上にプロジェクトを保存
- **マネージド環境**: サーバーの管理は Google が担当

### 2.3 V8 ランタイム vs Rhino

| 特徴 | V8 ランタイム（現行・推奨） | Rhino ランタイム（旧型） |
|------|---------------------------|------------------------|
| **JavaScript バージョン** | Modern ECMAScript (ES6+) | JavaScript 1.6 + 一部 1.7/1.8 機能 |
| **推奨度** | 強く推奨 | 非推奨 |
| **パフォーマンス** | 高速 | 比較的低速 |
| **モダン構文** | アロー関数、async/await、クラス等をサポート | 制限あり |

---

## 3. プロジェクト構成

### 3.1 GAS プロジェクトの構造

```
my-gas-project/
├── Code.gs              # メインのスクリプトファイル
├── appscript.json       # プロジェクトマニフェスト
└── (その他 .gs ファイル) # 追加のスクリプトファイル
```

### 3.2 ファイル構成

- **`.gs` ファイル**: Apps Script コードを含むサーバーサイド JavaScript ファイル
- **`appscript.json`**: プロジェクトの設定、権限、タイムゾーン等を定義するマニフェストファイル

### 3.3 スクリプトエディタ

- **組み込みエディタ**: ブラウザベースの Apps Script エディタ
- **オートコンプリート**: コンテンツアシスト機能により、グローバルオブジェクト、メソッド、列挙型を自動補完
- **リアルタイムデバッグ**: Logger.log() によるログ出力

---

## 4. 組み込みサービス

### 4.1 サービスアーキテクチャ

Apps Script は 30 以上の組み込みサービスを提供する。これらは JavaScript の標準 `Math` オブジェクトのようなグローバルオブジェクトとして提供される。

```javascript
// サービスの呼び出し例
GmailApp.sendEmail('recipient@example.com', 'Subject', 'Body');
SpreadsheetApp.openById('SPREADSHEET_ID');
DriveApp.createFolder('Folder Name');
```

### 4.2 主なサービスカテゴリ

| カテゴリ | サービス例 |
|---------|-----------|
| **Google Workspace Services** | GmailApp, SpreadsheetApp, DocumentApp, DriveApp, CalendarApp, FormsApp, SlidesApp |
| **Script Services** | Browser, Logger, MimeType, Session, Utilities |
| **Advanced Services** | BigQuery, Cloud Bigtable, Cloud Monitoring, etc. |

### 4.3 グローバルオブジェクト

各サービスは少なくとも1つのグローバル（トップレベル）オブジェクトを提供：

```javascript
// 単一オブジェクトのサービス
GmailApp        // Gmail サービス
SpreadsheetApp  // スプレッドシートサービス

// 複数オブジェクトを持つサービス（Base サービス）
Browser         // UI 関連
Logger          // ログ出力
MimeType        // MIME タイプ
Session         // セッション情報
```

---

## 5. 公式ドキュメントの構造

### 5.1 ドキュメントセクション

| セクション | 内容 |
|-----------|------|
| **Overview** | Apps Script の概要と基本概念 |
| **Guides** | 各機能の詳細ガイド |
| **Reference** | API リファレンス（サービス、クラス、メソッド） |
| **Samples** | コードサンプルとチュートリアル |
| **Support** | ヘルプ、バグ報告、機能リクエスト |

### 5.2 重要なドキュメントページ

- [Apps Script Overview](https://developers.google.com/apps-script)
- [Built-in Services Guide](https://developers.google.com/apps-script/guides/services)
- [V8 Runtime](https://developers.google.com/apps-script/guides/v8-runtime)
- [Best Practices](https://developers.google.com/apps-script/best_practices)

---

## 6. 推奨されるコードスタイル

### 6.1 命名規則

| 種類 | 規則 | 例 |
|------|------|-----|
| **変数** | camelCase | `userName`, `totalAmount` |
| **関数** | camelCase | `sendEmail()`, `calculateTotal()` |
| **クラス** | PascalCase | `UserService`, `DataProcessor` |
| **定数** | UPPER_SNAKE_CASE | `MAX_RETRIES`, `API_ENDPOINT` |
| **グローバルサービス** | PascalApp（命名規則固定） | `GmailApp`, `SpreadsheetApp` |

### 6.2 フォーマット

- **インデント**: 2 スペース（ Apps Script エディタのデフォルト）
- **行長**: 80 文字程度を目安に適切に改行
- **セミコロン**: 必須
- **引用符**: 一貫してシングルまたはダブルを使用

### 6.3 ベストプラクティス

```javascript
// 1. メソッドチェーン（推奨）
DocumentApp.create('New document')
    .getTab('t.0')
    .asDocumentTab()
    .getBody()
    .appendParagraph('New paragraph.');

// 2. 列挙型（Enums）の使用
folder.setSharing(DriveApp.Access.ANYONE, DriveApp.Permission.EDIT);

// 3. インターフェースのキャスト
var element = body.getChild(childIndex);
if (element.getType() == DocumentApp.ElementType.PARAGRAPH) {
    var paragraph = element.asParagraph();
    // 処理...
}

// 4. 適切な変数名
function sendWeeklyReport(emailAddress) {
    var subject = 'Weekly Report - ' + new Date();
    var body = generateReportBody();
    GmailApp.sendEmail(emailAddress, subject, body);
}
```

### 6.4 エラーハンドリング

```javascript
// try-catch を使用したエラーハンドリング
try {
    var sheet = SpreadsheetApp.openById(sheetId);
    // 処理...
} catch (e) {
    Logger.log('Error: ' + e.toString());
    // エラー処理...
}
```

---

## 7. JavaScript バージョン対応状況

### 7.1 V8 ランタイムでサポートされる機能（ES6+）

| 機能 | サポート |
|------|---------|
| **アロー関数** | ✅ `const add = (a, b) => a + b;` |
| **クラス** | ✅ `class UserService { ... }` |
| **let/const** | ✅ ブロックスコープの変数宣言 |
| **テンプレートリテラル** | ✅ `` `Hello, ${name}` `` |
| **分割代入** | ✅ `const { name, email } = user;` |
| **スプレッド演算子** | ✅ `const arr = [...array1, ...array2];` |
| **Promise / async-await** | ✅ 非同期処理 |
| **モジュール** | ⚠️ 一部制限あり |

### 7.2 注意点

- V8 ランタイムはデフォルトで有効
- 既存のプロジェクトで Rhino から V8 に移行する場合はコードの確認が必要
- 最新の ECMAScript 機能の多くがサポートされている

---

## 8. リソース

### 8.1 公式リソース

- **公式ドキュメント**: https://developers.google.com/apps-script
- **リファレンス**: https://developers.google.com/apps-script/reference
- **コミュニティ**: Stack Overflow (`google-apps-script` タグ)
- **ブログ**: Google Workspace Developers Blog

### 8.2 開発ツール

- **Apps Script Dashboard**: プロジェクト管理
- **REST API**: スクリプトプロジェクトのプログラム管理
- **clasp**: コマンドラインツール（ローカル開発）

---

*このリサーチドキュメントは Google Apps Script の基本情報とアーキテクチャに関する包括的な概要を提供するものである。*
