# GAS（Google Apps Script）の能力と制限

> **Version**: 1.0.0
> **Last Updated**: 2026-01-29
> **Source**: https://developers.google.com/apps-script/guides/services/quotas

## 概要

Google Apps Script（GAS）は、Google Workspaceサービスを自動化・拡張するためのクラウドベースのJavaScriptプラットフォームである。本ドキュメントでは、GASで可能な操作と制限を網羅的に記載する。

## できること

### 対応Googleサービス

#### Google Workspaceサービス

| サービス | 主な操作 | サービスオブジェクト |
|---------|----------|---------------------|
| **Gmail** | メール送受信、ドラフト作成、ラベル管理、検索 | `GmailApp` |
| **Google Sheets** | セル操作、グラフ作成、ピボットテーブル、書式設定 | `SpreadsheetApp` |
| **Google Docs** | ドキュメント作成、段落・テーブル操作、画像挿入 | `DocumentApp` |
| **Google Slides** | スライド作成、図形操作、テキスト編集 | `SlidesApp` |
| **Google Forms** | フォーム作成、回答収集、バリデーション設定 | `FormApp` |
| **Google Calendar** | イベント作成、カレンダー管理、招待 | `CalendarApp` |
| **Google Drive** | ファイル・フォルダ操作、権限管理、検索 | `DriveApp` |
| **Google Tasks** | タスク管理 | `TasksApp`（API経由） |
| **Google Chat** | メッセージ送信、カード作成 | `Chat API` |
| **Google Classroom** | コース管理、課題作成 | `Classroom API` |
| **Google Keep** | メモ操作 | （制限あり） |
| **Google Sites** | サイト操作 | （制限あり） |

#### その他のGoogleサービス

| サービス | 操作内容 |
|---------|----------|
| **Google Maps** | ルート検索、ジオコーディング、標高取得、静的マップ |
| **Google Translate** | 翻訳 |
| **YouTube** | 動画管理、Analytics |
| **BigQuery** | データクエリ実行 |
| **Analytics** | レポート取得 |

#### 外部連携

| 機能 | 詳細 |
|------|------|
| **URL Fetch** | HTTPリクエスト送信（外部API連携） |
| **JDBC** | 外部データベース接続（MySQL, PostgreSQL等） |
| **HTML Service** | Webアプリ公開、HTML/CSS/JS使用 |
| **Content Service** | JSON/XML/テキスト配信 |

### トリガー（自動実行）

#### シンプルトリガー

| トリガー | 関数名 | 説明 | 対応アプリ |
|---------|--------|------|-----------|
| オープン | `onOpen(e)` | ファイル開く時に実行 | Sheets, Docs, Slides, Forms |
| 編集 | `onEdit(e)` | セル編集時に実行 | Sheetsのみ |
| 選択変更 | `onSelectionChange(e)` | 選択変更時に実行 | Sheetsのみ |
| インストール | `onInstall(e)` | アドオンインストール時 | 各アプリ |
| HTTP GET | `doGet(e)` | WebアプリへのGETリクエスト | スタンドアロン |
| HTTP POST | `doPost(e)` | WebアプリへのPOSTリクエスト | スタンドアロン |

#### インストール可能トリガー

| トリガー | 説明 | 対応アプリ |
|---------|------|-----------|
| 時間駆動 | cronのように定期実行（1分〜1ヶ月） | 全て |
| オープン | ファイル開く時に実行（認証可） | Sheets, Docs, Forms |
| 編集 | セル編集時に実行（認証可） | Sheets |
| 変更 | スプレッドシート構造変更時 | Sheets |
| フォーム送信 | フォーム回答時 | Forms, Sheets |
| カレンダー更新 | カレンダーイベント変更時 | Calendar |

### Webアプリとしての公開

- **HTML Service**: 完全なWebアプリケーション構築可能
- **公開オプション**:
  - 自分のみアクセス可能
  - ドメイン内のユーザーのみ
  - 全員（認証済みユーザー）
  - 全員（匿名含む）
- **CORS対応**: Cross-Origin Resource Sharing対応

### コンテナバインド vs スタンドアロン

| タイプ | 説明 | メリット |
|-------|------|----------|
| **コンテナバインド** | Sheets/Docs/Forms等に紐付いたスクリプト | 親ファイルへの直接アクセス可、簡易トリガー使用可 |
| **スタンドアロン** | 独立したスクリプトファイル | 複数ファイル操作可、Webアプリに適している |

## できないこと・制限

### 実行時間制限

| タイプ | Consumer | Workspace | 備考 |
|-------|----------|-----------|------|
| スクリプト実行時間 | **6分/実行** | **6分/実行** | 超えると例外発生 |
| カスタム関数 | **30秒/実行** | **30秒/実行** | スプレッドシート内関数 |
| アドオン実行 | **30秒/実行** | **30秒/実行** | Workspace Studioアドオンは2分 |
| トリガー総実行時間 | **90分/日** | **6時間/日** | 全トリガーの合計 |

### クォータ（1日あたり）

#### Gmail関連

| 項目 | Consumer | Workspace |
|------|----------|-----------|
| メール送信（受信者数） | 100人/日 | 1,500人/日 |
| ドメイン内メール送信 | 100人/日 | 2,000人/日 |
| メール読み書き（送信除く） | 20,000/日 | 50,000/日 |
| 添付ファイル数/メール | 250個 | 250個 |
| 添付ファイル合計サイズ | 25MB | 25MB |
| メール本文サイズ | 200KB | 400KB |
| 受信者数/メール | 50人 | 50人 |

#### ドライブ・ファイル操作

| 項目 | Consumer | Workspace |
|------|----------|-----------|
| スプレッドシート作成 | 250/日 | 3,200/日 |
| ドキュメント作成 | 250/日 | 1,500/日 |
| プレゼンテーション作成 | 250/日 | 1,500/日 |
| スライド作成 | 250/日 | 1,500/日 |
| ファイル変換 | 2,000/日 | 4,000/日 |

#### カレンダー・連絡先

| 項目 | Consumer | Workspace |
|------|----------|-----------|
| カレンダーイベント作成 | 5,000/日 | 10,000/日 |
| 連絡先作成 | 1,000/日 | 2,000/日 |
| グループ読み取り | 2,000/日 | 10,000/日 |

#### 外部連携

| 項目 | Consumer | Workspace |
|------|----------|-----------|
| URL Fetch 呼び出し | 20,000/日 | 100,000/日 |
| URL Fetch レスポンスサイズ | 50MB/コール | 50MB/コール |
| URL Fetch POSTサイズ | 50MB/コール | 50MB/コール |
| URL Fetch URL長 | 2KB/コール | 2KB/コール |
| URL Fetch ヘッダー数 | 100/コール | 100/コール |
| JDBC接続 | 10,000/日 | 50,000/日 |
| JDBC失敗接続 | 100/日 | 500/日 |

#### Mapsサービス

| 項目 | Consumer | Workspace |
|------|----------|-----------|
| Static Map レンダリング | 1,000/日 | 10,000/日 |
| ルートクエリ | 1,000/日 | 10,000/日 |
| ジオコーディング | 1,000/日 | 10,000/日 |
| 標高サンプル | 1,000/日 | 10,000/日 |

#### その他

| 項目 | Consumer | Workspace |
|------|----------|-----------|
| Translate 呼び出し | 5,000/日 | 20,000/日 |
| Properties 読み書き | 50,000/日 | 500,000/日 |
| GASプロジェクト作成 | 50/日 | 50/日 |

### 同時実行制限

| 項目 | 制限 |
|------|------|
| ユーザーあたり同時実行 | **30** |
| スクリプトあたり同時実行 | **1,000** |

### トリガー制限

| 項目 | 制限 |
|------|------|
| トリガー数（ユーザー/スクリプト） | **20個** |
| プロジェクトバージョン数 | **200** |

### シンプルトリガーの追加制限

シンプルトリガー（`onOpen`, `onEdit`等）には以下の制限がある：

1. **認証不要で実行される** - ユーザー権限で自動実行
2. **認証が必要なサービスにアクセス不可**:
   - Gmail送信
   - カレンダー操作
   - Driveへのファイル保存等
3. **実行時間は30秒に制限**
4. **読み取りモードでは実行されない**
5. **スクリプト実行やAPIリクエストではトリガーされない**
6. **バインドされたファイルのみ操作可能**（他ファイルへのアクセス不可）

### UIスレッドでの実行制限

- **Alert/Promptは使用可能**: `Browser.msgBox()`, `Browser.inputBox()`
- **ただし、Webアプリやトリガーでは動作しない**
- **これらの関数はUIスレッドで実行する必要がある**

### 認証に関する制限

- インストール可能トリガーは**常に作成者のアカウントで実行**
- 他のユーザーがファイルを開いても、トリガーは作成者の権限で動作
- 異なるアカウントのトリガーは互いに見えない

### 外部API連携の制限

- **CORS**: UrlFetchAppはCORS制限を受けない（サーバーサイド）
- **認証**: OAuth 2.0対応可能
- **HTTPS**: 推奨（HTTPも可能）
- **タイムアウト**: 特に明記なし（実行時間制限内）

## 例外メッセージ

クォータ超過時のメッセージ例：

| メッセージ | 原因 |
|-----------|------|
| `Limit exceeded: Email Attachments Per Message.` | 添付ファイル数超過 |
| `Service invoked too many times: Calendar.` | サービス呼び出し回数超過（1日） |
| `Service invoked too many times in a short time: Calendar.` | 短時間に多数の呼び出し |
| `Service using too much computer time for one day.` | 1日の実行時間超過 |
| `Script invoked too many times per second for this Google user account.` | 1秒間の呼び出し回数超過 |
| `There are too many scripts running simultaneously for this Google user account.` | 同時実行数超過 |

## ベストプラクティス

### クォータ管理

1. **Utilities.sleep()を使用**: 短時間の多数呼び出しを回避
2. **バッチ処理**: 複数の操作をまとめて実行
3. **キャッシュ利用**: CacheServiceで頻繁アクセスデータを保存
4. **トリガー分散**: 時間駆動トリガーで負荷分散

### 実行時間対策

1. **6分以内に完了する設計**
2. **長時間処理はトリガーで分割**
3. **不要なループを避ける**
4. **API呼び出しを最小化**

### エラーハンドリング

```javascript
try {
  // 処理
} catch (e) {
  if (e.message.includes('Service invoked too many times')) {
    Utilities.sleep(1000);
    // リトライ
  }
}
```

## 参考資料

- [Quotas for Google Services](https://developers.google.com/apps-script/guides/services/quotas)
- [Simple Triggers](https://developers.google.com/apps-script/guides/triggers)
- [Installable Triggers](https://developers.google.com/apps-script/guides/triggers/installable)
- [Web Apps](https://developers.google.com/apps-script/guides/web)
- [Best Practices](https://developers.google.com/apps-script/guides/support/best-practices)
