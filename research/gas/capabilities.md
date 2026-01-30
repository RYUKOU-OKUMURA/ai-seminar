# Google Apps Script（GAS）能力調査レポート

> **Version**: 2.0.0
> **Last Updated**: 2026-01-29
> **Source**: [Quotas for Google Services](https://developers.google.com/apps-script/guides/services/quotas)

## 概要

本レポートは、Google Apps Script（GAS）の技術的能力、制限、およびベストプラクティスについて包括的にまとめたものです。2025年〜2026年の最新情報に基づき、開発者がGASを効果的に活用するための詳細なガイドを提供します。

---

## GASでできること（可能な操作、各Googleサービスの操作）

### 1. Google Workspaceサービスの操作

#### 1.1 Google Sheets（スプレッドシート）

**可能な操作:**
- セルの読み取り、書き込み、フォーマット
- カスタム関数の作成（`=MYFUNCTION()`のような数式）
- カスタムメニュー、ダイアログ、サイドバーの追加
- グラフとチャートの埋め込み
- データ検証ルールの設定
- ピボットテーブルの作成と操作
- トリガーによる自動化（`onOpen`、`onEdit`、`onSelectionChange`）
- Google Formsとの連携

**主なクラス:**
- `SpreadsheetApp` - スプレッドシート操作のメインサービス
- `Sheet` - シート操作
- `Range` - セル範囲操作
- `Charts` - グラフ作成

**使用例:**
```javascript
function logProductInfo() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var data = sheet.getDataRange().getValues();
  for (var i = 0; i < data.length; i++) {
    Logger.log('Product name: ' + data[i][0]);
    Logger.log('Product number: ' + data[i][1]);
  }
}
```

#### 1.2 Google Docs（ドキュメント）

**可能な操作:**
- ドキュメントコンテンツの読み取りと編集
- 段落、テーブル、画像の挿入と操作
- カスタムメニューの追加
- ブックマークと名前付き範囲の管理

**主なクラス:**
- `DocumentApp` - ドキュメント操作のメインサービス
- `Body` - ドキュメント本文
- `Paragraph` - 段落操作
- `Table` - テーブル操作

#### 1.3 Google Slides（スライド）

**可能な操作:**
- スライドの作成と編集
- 図形、画像、テキストの操作
- テーマとレイアウトの管理
- スライドの追加と削除

**主なクラス:**
- `SlidesApp` - スライド操作のメインサービス
- `Slide` - 個別スライド
- `PageElement` - スライド内の要素
- `Shape` - 図形操作

#### 1.4 Google Forms（フォーム）

**可能な操作:**
- フォームの作成と編集
- 質問項目の追加と設定
- フォームレスポンスの取得と分析
- スプレッドシートとの自動連携
- カスタム検証ルールの設定

**主なクラス:**
- `FormApp` - フォーム操作のメインサービス
- `Form` - フォームオブジェクト
- `FormResponse` - レスポンスデータ

#### 1.5 Gmail

**可能な操作:**
- メールの送信（`MailApp`、`GmailApp`）
- メールの読み取りと検索
- ラベルの管理
- スレッド操作
- 下書きの作成と管理

**制限（Consumerアカウント）:**
- 1日あたり100宛先まで（2025年最新）
- 1通あたり最大50宛先
- 添付ファイルサイズは合計25MBまで

**主なクラス:**
- `GmailApp` - Gmail操作のメインサービス
- `GmailMessage` - メールメッセージ
- `GmailThread` - スレッド
- `GmailLabel` - ラベル

#### 1.6 Google Calendar

**可能な操作:**
- カレンダーイベントの作成、更新、削除
- 複数カレンダーの管理
- イベント招待の送信
- 定期的なイベントの作成

**制限:**
- Consumer: 5,000イベント/日
- Google Workspace: 10,000イベント/日

**主なクラス:**
- `CalendarApp` - カレンダー操作のメインサービス
- `Calendar` - カレンダーオブジェクト
- `CalendarEvent` - イベント

#### 1.7 Google Drive

**可能な操作:**
- ファイルとフォルダの作成、移動、削除
- 権限管理（共有設定）
- ファイルメタデータの取得と更新
- フォルダ構造のナビゲーション

**主なクラス:**
- `DriveApp` - Drive操作のメインサービス
- `Folder` - フォルダオブジェクト
- `File` - ファイルオブジェクト

#### 1.8 その他のGoogleサービス

| サービス | 操作内容 | サービスオブジェクト |
|---------|----------|---------------------|
| **Google Classroom** | コースの作成と管理、課題の作成と採点、生徒の招待と管理 | `Classroom API` |
| **Google Tasks** | タスクリストの管理、タスクの作成と更新 | `TasksApp`（API経由） |
| **Google Translate** | テキスト翻訳、言語検出 | `LanguageApp` |
| **Google Maps** | ジオコーディング（住所→座標）、ルート検索、静的マップの生成 | `Maps` |
| **YouTube** | 動画情報の取得、プレイリスト管理、アップロード（高度な設定が必要） | `YouTube Data API` |
| **BigQuery** | BigQueryクエリの実行、データのエクスポートとインポート、テーブル操作 | `BigQuery API` |
| **Analytics** | レポートの取得 | `Analytics Data API` |
| **Google Chat** | メッセージ送信、カード作成 | `Chat API` |
| **Google Keep** | メモ操作 | （制限あり） |
| **Google Sites** | サイト操作 | （制限あり） |

### 2. 外部サービス連携

#### 2.1 URL Fetchサービス（UrlFetchApp）

**可能な操作:**
- HTTP/HTTPSリクエストの送信（GET、POST、PUT、DELETE等）
- 外部APIとの連携
- JSON/XMLデータの取得と解析
- 認証付きリクエスト（OAuth、APIキー）

**使用例:**
```javascript
function fetchExternalAPI() {
  var response = UrlFetchApp.fetch('https://api.example.com/data', {
    method: 'GET',
    headers: {
      'Authorization': 'Bearer ' + getAccessToken()
    },
    muteHttpExceptions: true
  });
  var json = JSON.parse(response.getContentText());
  return json;
}
```

#### 2.2 JDBCサービス

**可能な操作:**
- 外部データベースへの接続（MySQL、PostgreSQL、Oracle、SQL Server等）
- SQLクエリの実行
- トランザクション管理

**制限:**
- Consumer: 10,000接続/日、100失敗接続/日
- Google Workspace: 50,000接続/日、500失敗接続/日

#### 2.3 BigQuery連携

**可能な操作:**
- BigQueryクエリの実行
- データのエクスポートとインポート
- テーブル操作

### 3. ユーティリティサービス

#### 3.1 HTMLサービス（HtmlService）

**可能な操作:**
- Webアプリの作成
- ダイアログとサイドバーの表示
- クライアント側JavaScriptとの連携（`google.script.run`）

#### 3.2 コンテンツサービス（ContentService）

**可能な操作:**
- JSON、XML、RSS、テキストフィードの出力
- Web APIエンドポイントの作成

#### 3.3 キャッシュサービス（CacheService）

**可能な操作:**
- データの一時保存
- 高速なデータアクセス

#### 3.4 プロパティサービス（PropertiesService）

**可能な操作:**
- スクリプトプロパティ、ユーザープロパティ、ドキュメントプロパティの保存
- 設定値の永続化

**制限:**
- 値の最大サイズ: 9KB
- 合計ストレージ: 500KB/プロパティストア

#### 3.5 ロックサービス（LockService）

**可能な操作:**
- 排他制御
- 同時実行の管理

### 4. 高度な機能

#### 4.1 アドオンの開発

**可能な操作:**
- Google Workspace Marketplaceでの公開
- エディタアドオン（Sheets、Docs、Slides等）
- Gmailアドオン

#### 4.2 Webアプリとしての公開

**可能な操作:**
- 独立したWebアプリケーションとしてデプロイ
- RESTful APIの作成（`doGet`、`doPost`）
- Google Sitesへの埋め込み

#### 4.3 ライブラリの作成と共有

**可能な操作:**
- 再利用可能なコードライブラリの作成
- 他のプロジェクトからのインポート
- バージョン管理

---

## GASの制限（実行時間、メモリ、クォータ）

### 1. 実行時間の制限

| 実行タイプ | Consumerアカウント | Google Workspaceアカウント |
|-----------|-------------------|---------------------------|
| 通常スクリプト | 6分/実行 | 6分/実行 |
| カスタム関数 | 30秒/実行 | 30秒/実行 |
| Workspaceアドオン | 30秒/実行 | 30秒/実行 |

**重要事項:**
- 6分（360秒）を超えると「Exception: Service error: SpreadsheetsApp」などのエラーが発生
- 長時間実行が必要な場合は、処理を分割してトリガーで継続する必要あり
- カスタム関数は30秒制限が厳しいため、複雑な計算には不向き

### 2. 1日クォータ（2025年最新）

| 機能 | Consumer（gmail.com等） | Google Workspace |
|------|------------------------|------------------|
| カレンダーイベント作成 | 5,000/日 | 10,000/日 |
| 連絡先作成 | 1,000/日 | 2,000/日 |
| ドキュメント作成 | 250/日 | 1,500/日 |
| ファイル変換 | 2,000/日 | 4,000/日 |
| **メール送信宛先** | **100/日** | **1,500/日** |
| ドメイン内メール送信宛先 | 100/日 | 2,000/日 |
| メール読み書き（送信除く） | 20,000/日 | 50,000/日 |
| グループ読み取り | 2,000/日 | 10,000/日 |
| **JDBC接続** | **10,000/日** | **50,000/日** |
| **JDBC失敗接続** | **100/日** | **500/日** |
| プレゼンテーション作成 | 250/日 | 1,500/日 |
| プロパティ読み書き | 50,000/日 | 500,000/日 |
| スライド作成 | 250/日 | 1,500/日 |
| **スプレッドシート作成** | **250/日** | **3,200/日** |
| **トリガー総実行時間** | **90分/日** | **6時間/日** |
| **URL Fetch呼び出し** | **20,000/日** | **100,000/日** |
| 静的マップレンダリング | 1,000/日 | 10,000/日 |
| Google Mapルートクエリ | 1,000/日 | 10,000/日 |
| Google Mapジオコード呼び出し | 1,000/日 | 10,000/日 |
| 翻訳呼び出し | 5,000/日 | 20,000/日 |
| Google Map標高サンプルクエリ | 1,000/日 | 10,000/日 |
| Apps Scriptプロジェクト作成 | 50/日 | 50/日 |

**注:**
- クォータは最初のリクエストから24時間後にリセット
- Google Workspaceは従来のG Suite無料版（廃止）も含む
- トライアルアカウントには追加の制限あり

### 3. 技術的制限

| 項目 | 制限 |
|------|------|
| スクリプトランタイム | 6分/実行 |
| カスタム関数ランタイム | 30秒/実行 |
| Workspaceアドオンランタイム | 30秒/実行 |
| **同時実行数（ユーザーあたり）** | **30** |
| **同時実行数（スクリプトあたり）** | **1,000** |
| メール添付ファイル | 250/メッセージ |
| メール本文サイズ | 200KB/メッセージ（Consumer）<br>400KB/メッセージ（Workspace） |
| メール受信者数/メッセージ | 50/メッセージ |
| メール添付ファイル合計サイズ | 25MB/メッセージ |
| プロパティ値サイズ | 9KB/値 |
| プロパティ合計ストレージ | 500KB/プロパティストア |
| **トリガー数** | **20/ユーザー/スクリプト** |
| URL Fetchレスポンスサイズ | 50MB/呼び出し |
| URL Fetchヘッダー数 | 100/呼び出し |
| URL Fetchヘッダーサイズ | 8KB/呼び出し |
| URL Fetch POSTサイズ | 50MB/呼び出し |
| URL Fetch URL長 | 2KB/呼び出し |
| バージョン数 | 200/スクリプト |

### 4. メモリとリソースの制限

- **メモリ:** 公式数値は非公開ですが、実用的には数百MB程度
- **一時ストレージ:** 実行終了後にクリアされる
- **ファイルサイズ上限:**
  - スクリプトファイル: 1ファイルあたり数MB
  - プロジェクト全体: 明示的な制限なし（ただし実用的には数十MB）

### 5. URL Fetchサービスの制限

| 項目 | Consumer | Google Workspace |
|------|----------|------------------|
| 1日あたりのリクエスト数 | 20,000 | 100,000 |
| レスポンスサイズ | 50MB/呼び出し | 50MB/呼び出し |
| POSTデータサイズ | 50MB/呼び出し | 50MB/呼び出し |
| URL長 | 2KB | 2KB |
| ヘッダー数 | 100/呼び出し | 100/呼び出し |
| ヘッダーサイズ | 8KB/呼び出し | 8KB/呼び出し |

**IPアドレスに関する重要事項:**
- すべてのUrlFetchAppリクエストはGoogleのサーバーから発信
- Googleはリージョンごとに固定IPセットを使用
- 複数ユーザーが同じGoogle IPを共有するため、外部APIのレート制限に影響

### 6. トリガー総実行時間の制限

| アカウントタイプ | 1日あたりの総実行時間 |
|-----------------|---------------------|
| Consumer | 90分/日 |
| Google Workspace | 6時間/日 |

**対策:**
- 短時間で終わる処理に分割
- 不要なトリガーを削除
- 効率的なアルゴリズムの実装

---

## GASでできないこと（UIスレッド制限等）

### 1. シンプルトリガーのUI制限

シンプルトリガー（`onOpen(e)`、`onEdit(e)`、`onSelectionChange(e)`等）は以下の制限があります：

#### 1.1 使用できないUIメソッド

シンプルトリガーでは以下のUI操作が**できません**：

```javascript
// これらはシンプルトリガーでは動作しない
Browser.msgBox("メッセージ");           // × エラー
Browser.inputBox("入力してください");   // × エラー
SpreadsheetApp.getUi().alert("警告");   // × エラー
SpreadsheetApp.getUi().prompt("確認");  // × エラー
SpreadsheetApp.getUi().showSidebar();   // × エラー
SpreadsheetApp.getUi().showModalDialog(); // × エラー
```

**エラーメッセージ例:**
```
Exception: You do not have permission to call SpreadsheetApp.getUi()
Exception: Cannot call SpreadsheetApp.getUi() from this context
```

#### 1.2 シンプルトリガーの制限事項

| 制限項目 | 詳細 |
|---------|------|
| **認証不要で実行** | ユーザー明示的な許可なしで自動実行される |
| **認証が必要なサービスにアクセス不可** | Gmail送信、Calendar操作等が不可 |
| **UIの表示が不可** | `alert`、`prompt`、`showSidebar`等が使用不可 |
| **他ファイルへのアクセス不可** | バインドされているファイル以外にアクセス不可 |
| **実行時間は30秒まで** | 6分ではなく30秒制限 |
| **ユーザー識別の制限** | セキュリティ制限によりユーザー識別が制限される |
| **読み取り専用モードでは実行されない** | 閲覧モードやコメントモードでは実行されない |
| **スクリプト実行やAPI呼び出しでは起動しない** | プログラム的な編集では`onEdit`が起動しない |

#### 1.3 シンプルトリガーでできること

```javascript
// カスタムメニューの追加（可能）
function onOpen(e) {
  SpreadsheetApp.getUi()
    .createMenu('カスタムメニュー')
    .addItem('項目1', 'menuItem1')
    .addToUi();
}

// セルの変更（可能）
function onEdit(e) {
  var range = e.range;
  range.setNote('最終更新: ' + new Date());
}

// 選択範囲の変更（可能）
function onSelectionChange(e) {
  var range = e.range;
  if (range.getValue() === '') {
    range.setBackground('red');
  }
}
```

### 2. 認証が必要な操作の制限

シンプルトリガーでは以下の操作が**できません**：

#### 2.1 Gmail関連

```javascript
function onOpen(e) {
  // これらはすべてエラーになる
  GmailApp.sendEmail('test@example.com', '件名', '本文'); // ×
  MailApp.sendEmail('test@example.com', '件名', '本文');  // ×
}
```

#### 2.2 Calendar関連

```javascript
function onEdit(e) {
  // エラーになる
  CalendarApp.createEvent('イベント名', ...); // ×
}
```

#### 2.3 外部API呼び出し

```javascript
function onSelectionChange(e) {
  // エラーになる（UrlFetchAppも使用不可）
  UrlFetchApp.fetch('https://api.example.com'); // ×
}
```

### 3. 解決策

#### 3.1 インストール可能トリガーの使用

インストール可能トリガーを使えば、以下が可能になります：

```javascript
// インストール可能トリガーならUI表示が可能
function installableOnEdit(e) {
  var ui = SpreadsheetApp.getUi();
  ui.alert('セルが編集されました'); // ○ 動作する
  GmailApp.sendEmail(...);           // ○ 動作する
}

// トリガーをプログラムで作成
function createTrigger() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  ScriptApp.newTrigger('installableOnEdit')
    .forSpreadsheet(ss)
    .onEdit()
    .create();
}
```

#### 3.2 メニュー項目からの実行

シンプルトリガーでメニューを作成し、メニュークリック時に関数を実行：

```javascript
function onOpen(e) {
  SpreadsheetApp.getUi()
    .createMenu('ツール')
    .addItem('通知を送信', 'sendNotification')
    .addToUi();
}

// メニューから呼び出される関数は認証済みとして実行される
function sendNotification() {
  SpreadsheetApp.getUi().alert('通知を送信します');
  GmailApp.sendEmail(...); // ○ 動作する
}
```

### 4. その他の制限事項

#### 4.1 UIスレッドでの実行制限

- シンプルトリガーは「認証不要モード」で実行される
- UIを表示する操作はすべて「認証済みモード」が必要
- このため、シンプルトリガーからUIを表示しようとするとエラー

#### 4.2 編集トリガーのキュー制限

```javascript
// onEditトリガーは最大2つのイベントしかキューに入らない
// 複数のセルを素早く編集すると、一部のイベントが失われる可能性がある
function onEdit(e) {
  // 複数回の編集があった場合、最初と最後のイベントのみ実行される
  var range = e.range;
  range.setNote('編集されました');
}
```

#### 4.3 選択変更トリガーの制限

```javascript
// onSelectionChangeは2秒以内に複数回発生した場合、
// 最初と最後のイベントのみ実行される
function onSelectionChange(e) {
  // 頻繁な選択変更はスキップされる可能性がある
  var range = e.range;
  range.setBackground('yellow');
}
```

---

## Trigger（トリガー）の種類と制限

### 1. シンプルトリガー

#### 1.1 種類

| トリガー名 | 関数名 | 説明 | 対応アプリ |
|-----------|--------|------|-----------|
| Open | `onOpen(e)` | ファイルを開いた時に実行 | Sheets, Slides, Forms, Docs |
| Edit | `onEdit(e)` | セルを編集した時に実行 | Sheets |
| Selection Change | `onSelectionChange(e)` | 選択範囲を変更した時に実行 | Sheets |
| Install | `onInstall(e)` | アドオンをインストールした時に実行 | Sheets, Slides, Forms, Docs |
| Get | `doGet(e)` | WebアプリにHTTP GETリクエスト | Standalone |
| Post | `doPost(e)` | WebアプリにHTTP POSTリクエスト | Standalone |

#### 1.2 制限事項

- **認証不要で自動実行**
- **認証が必要なサービスにアクセス不可**
- **UIの表示が不可**（`alert`、`prompt`、`showSidebar`等）
- **他ファイルへのアクセス不可**
- **実行時間は30秒まで**
- **ユーザー識別の制限**
- **読み取り専用モードでは実行されない**
- **スクリプト実行やAPI呼び出しでは起動しない**

#### 1.3 シンプルトリガーのイベントオブジェクト

```javascript
function onEdit(e) {
  // イベントオブジェクトのプロパティ
  var range = e.range;        // 編集された範囲
  var value = e.value;        // 新しい値（文字列）
  var oldValue = e.oldValue;  // 古い値（文字列）
  var user = e.user;          // 編集したユーザー（取得できる場合）
}
```

### 2. インストール可能トリガー

#### 2.1 種類

| イベントタイプ | 説明 | 対応アプリ |
|---------------|------|-----------|
| Open | ファイルを開いた時に実行 | Sheets, Docs, Forms |
| Edit | セルを編集した時に実行 | Sheets |
| Change | スプレッドシートの内容が変更された時に実行 | Sheets |
| Form Submit | フォームが送信された時に実行 | Sheets, Forms |
| Time-driven | 時間ベースで実行（毎時、毎日、毎週等） | Sheets, Slides, Forms, Docs, Standalone |

#### 2.2 インストール可能トリガーの作成方法

**方法1: プログラムで作成**

```javascript
function createTimeDrivenTrigger() {
  // 毎時間実行
  ScriptApp.newTrigger('myFunction')
    .timeBased()
    .everyHours(1)
    .create();

  // 毎日午前6時に実行
  ScriptApp.newTrigger('myFunction')
    .timeBased()
    .atHour(6)
    .everyDays(1)
    .create();

  // 特定の日時に実行（一度のみ）
  ScriptApp.newTrigger('myFunction')
    .timeBased()
    .at(new Date('2025-12-31T23:59:00'))
    .create();
}

function createSpreadsheetEditTrigger() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  ScriptApp.newTrigger('myEditFunction')
    .forSpreadsheet(ss)
    .onEdit()
    .create();
}

function createFormSubmitTrigger() {
  var form = FormApp.openById('FORM_ID');
  ScriptApp.newTrigger('myFormSubmitFunction')
    .forUser(FormApp.getActiveUser())
    .onFormSubmit()
    .create();
}
```

**方法2: Apps Scriptエディタから手動で作成**

1. Apps Scriptエディタを開く
2. 左側の「トリガー」（時計アイコン）をクリック
3. 「トリガーを追加」をクリック
4. 設定項目を入力:
   - 実行する関数を選択
   - イベントのソースを選択（スプレッドシートから、時間主導型等）
   - イベントの種類を選択（編集時、フォーム送信時等）

#### 2.3 インストール可能トリガーの利点

- **認証が必要なサービスにアクセス可能**
- **UIの表示が可能**
- **他ファイルへのアクセス可能**
- **6分まで実行可能**
- **詳細なイベント情報を取得可能**

#### 2.4 Changeトリガー（onEditとの違い）

```javascript
function onChange(e) {
  // Changeトリガーは編集以外の変更も検知
  // 例: 行・列の追加/削除、セルの書式変更等
  var changeType = e.changeType;
  // changeTypeの値:
  // - EDIT: セル値の変更
  // - INSERT_ROW: 行の挿入
  // - INSERT_COLUMN: 列の挿入
  // - REMOVE_ROW: 行の削除
  // - REMOVE_COLUMN: 列の削除
  // - INSERT_GRID: シートの追加
  // - REMOVE_GRID: シートの削除
  // - FORMAT: セルの書式変更
  // - OTHER: その他の変更
}
```

### 3. トリガーの制限

| 項目 | 制限 |
|------|------|
| **トリガー総数** | 20/ユーザー/スクリプト |
| **1日のトリガー総実行時間** | Consumer: 90分/日<br>Google Workspace: 6時間/日 |
| **シンプルトリガー実行時間** | 30秒/実行 |
| **インストール可能トリガー実行時間** | 6分/実行 |

### 4. トリガーの管理

#### 4.1 トリガーの削除

```javascript
function deleteAllTriggers() {
  var triggers = ScriptApp.getProjectTriggers();
  for (var i = 0; i < triggers.length; i++) {
    ScriptApp.deleteTrigger(triggers[i]);
  }
}

function deleteSpecificTrigger(triggerId) {
  var triggers = ScriptApp.getProjectTriggers();
  for (var i = 0; i < triggers.length; i++) {
    if (triggers[i].getUniqueId() === triggerId) {
      ScriptApp.deleteTrigger(triggers[i]);
      break;
    }
  }
}
```

#### 4.2 トリガー情報の取得

```javascript
function listTriggers() {
  var triggers = ScriptApp.getProjectTriggers();
  for (var i = 0; i < triggers.length; i++) {
    var trigger = triggers[i];
    Logger.log('関数名: ' + trigger.getHandlerFunction());
    Logger.log('トリガーソース: ' + trigger.getTriggerSource());
    Logger.log('イベントタイプ: ' + trigger.getTriggerSourceId());
    Logger.log('ユニークID: ' + trigger.getUniqueId());
  }
}
```

### 5. ベストプラクティス

#### 5.1 シンプルトリガー vs インストール可能トリガー

| 項目 | シンプルトリガー | インストール可能トリガー |
|------|-----------------|----------------------|
| 設定の手間 | 不要（関数名だけで自動実行） | 必要（明示的に作成） |
| 認証 | 不要 | 必要 |
| Gmail等へのアクセス | 不可 | 可能 |
| UI表示 | 不可 | 可能 |
| 実行時間 | 30秒 | 6分 |
| 他ファイルアクセス | 不可 | 可能 |

#### 5.2 トリガーの選択ガイド

**シンプルトリガーを使用する場合:**
- メニューの追加
- 単純なセル編集のログ
- 書式の自動設定
- バインドされているファイル内のみの操作

**インストール可能トリガーを使用する場合:**
- GmailやCalendar等、認証が必要な操作
- ダイアログやサイドバーの表示
- 他のファイルへのアクセス
- 長時間実行される処理
- 複雑な条件分岐を伴う処理

---

## 外部API連携の可否と制限

### 1. 可能な外部API連携

#### 1.1 URL Fetchサービス（UrlFetchApp）

**基本機能:**
- HTTP/HTTPSプロトコルでのリクエスト送信
- 主要HTTPメソッドのサポート（GET、POST、PUT、DELETE、PATCH）
- リクエストヘッダーのカスタマイズ
- 認証付きリクエスト（Basic Auth、Bearer Token、API Key等）
- JSON/XML/テキストデータの送受信
- ファイルアップロード（multipart/form-data）
- リダイレクトの追跡
- タイムアウト設定

**使用例:**

```javascript
// GETリクエスト
function getDataFromAPI() {
  var url = 'https://api.example.com/data';
  var response = UrlFetchApp.fetch(url);

  if (response.getResponseCode() === 200) {
    var json = JSON.parse(response.getContentText());
    return json;
  }
}

// POSTリクエスト（JSON）
function postDataToAPI() {
  var url = 'https://api.example.com/data';
  var payload = {
    name: 'John Doe',
    email: 'john@example.com'
  };

  var options = {
    method: 'post',
    contentType: 'application/json',
    payload: JSON.stringify(payload),
    headers: {
      'Authorization': 'Bearer ' + getAccessToken()
    },
    muteHttpExceptions: true
  };

  var response = UrlFetchApp.fetch(url, options);
  return JSON.parse(response.getContentText());
}

// Basic認証
function basicAuthExample() {
  var url = 'https://api.example.com/secure';
  var username = 'user';
  var password = 'pass';

  var options = {
    headers: {
      'Authorization': 'Basic ' + Utilities.base64Encode(username + ':' + password)
    }
  };

  var response = UrlFetchApp.fetch(url, options);
  return response.getContentText();
}

// APIキー認証
function apiKeyAuthExample() {
  var url = 'https://api.example.com/data?key=' + getApiKey();
  var response = UrlFetchApp.fetch(url);
  return response.getContentText();
}
```

#### 1.2 OAuth 2.0認証

**対応可能なOAuth 2.0フロー:**
- サービスアカウントフロー（JWT署名付きリクエスト）
- アクセストークンの使用（事前に取得したトークン）

```javascript
// OAuth 2.0 アクセストークンを使用
function oauth2Example() {
  var accessToken = getValidAccessToken(); // 事前に取得したトークン
  var url = 'https://api.example.com/protected';

  var options = {
    headers: {
      'Authorization': 'Bearer ' + accessToken
    }
  };

  var response = UrlFetchApp.fetch(url, options);
  return response.getContentText();
}
```

#### 1.3 Webhookエンドポイントの作成

`doPost(e)`関数を使用して、Webhookエンドポイントを作成できます：

```javascript
// Webhookエンドポイント
function doPost(e) {
  var postData = JSON.parse(e.postData.contents);

  // 受信データの処理
  if (postData.event === 'payment.completed') {
    processPayment(postData.data);
  }

  return ContentService.createTextOutput(JSON.stringify({
    status: 'success',
    message: 'Webhook received'
  })).setMimeType(ContentService.MimeType.JSON);
}
```

### 2. 外部API連携の制限

#### 2.1 クォータ制限

| 項目 | Consumer | Google Workspace |
|------|----------|------------------|
| 1日あたりのリクエスト数 | 20,000 | 100,000 |
| レスポンスサイズ | 50MB/呼び出し | 50MB/呼び出し |
| POSTデータサイズ | 50MB/呼び出し | 50MB/呼び出し |
| URL長 | 2KB | 2KB |
| ヘッダー数 | 100/呼び出し | 100/呼び出し |
| ヘッダーサイズ | 8KB/呼び出し | 8KB/呼び出し |

#### 2.2 IPアドレスに関する制限

**重要事項:**
- すべてのUrlFetchAppリクエストはGoogleのサーバーから発信
- Googleはリージョンごとに固定IPセットを使用
- 複数ユーザーが同じGoogle IPを共有するため、外部APIのレート制限に影響

**対策:**
- APIプロバイダーにGoogle Apps Scriptからのアクセスを通知
- レート制限の緩和を申請
- APIキーまたは認証トークンを使用

#### 2.3 TLS/SSL証明書の制限

- 自己署名証明書は使用不可
- 信頼された認証局が発行した証明書のみ対応
- HTTPS接続のみ可能

#### 2.4 サポートされない機能

- WebSocket接続
- Server-Sent Events（SSE）
- FTP、SSH等のプロトコル
- 生のTCPソケット接続

### 3. よく使用されるAPIとの連携例

#### 3.1 REST API

```javascript
function callRestAPI() {
  var url = 'https://api.example.com/v1/users';
  var response = UrlFetchApp.fetch(url, {
    method: 'get',
    headers: {
      'Accept': 'application/json',
      'Authorization': 'Bearer ' + getAccessToken()
    }
  });

  return JSON.parse(response.getContentText());
}
```

#### 3.2 GraphQL API

```javascript
function callGraphQLAPI() {
  var url = 'https://api.example.com/graphql';
  var query = `
    query {
      user(id: "123") {
        name
        email
      }
    }
  `;

  var response = UrlFetchApp.fetch(url, {
    method: 'post',
    contentType: 'application/json',
    payload: JSON.stringify({
      query: query
    }),
    headers: {
      'Authorization': 'Bearer ' + getAccessToken()
    }
  });

  return JSON.parse(response.getContentText());
}
```

#### 3.3 Slack Webhook

```javascript
function sendSlackNotification(message) {
  var webhookUrl = 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL';
  var payload = {
    text: message,
    username: 'GAS Bot',
    icon_emoji: ':robot_face:'
  };

  UrlFetchApp.fetch(webhookUrl, {
    method: 'post',
    contentType: 'application/json',
    payload: JSON.stringify(payload)
  });
}
```

#### 3.4 Discord Webhook

```javascript
function sendDiscordNotification(message) {
  var webhookUrl = 'https://discord.com/api/webhooks/YOUR/WEBHOOK/URL';
  var payload = {
    content: message
  };

  UrlFetchApp.fetch(webhookUrl, {
    method: 'post',
    contentType: 'application/json',
    payload: JSON.stringify(payload)
  });
}
```

#### 3.5 LINE Messaging API

```javascript
function sendLineMessage(userId, message) {
  var url = 'https://api.line.me/v2/bot/message/push';
  var channelAccessToken = 'YOUR_CHANNEL_ACCESS_TOKEN';

  var payload = {
    to: userId,
    messages: [
      {
        type: 'text',
        text: message
      }
    ]
  };

  UrlFetchApp.fetch(url, {
    method: 'post',
    contentType: 'application/json',
    headers: {
      'Authorization': 'Bearer ' + channelAccessToken
    },
    payload: JSON.stringify(payload)
  });
}
```

### 4. エラーハンドリング

```javascript
function fetchWithErrorHandling(url) {
  var maxRetries = 3;
  var retryDelay = 1000; // 1秒

  for (var i = 0; i < maxRetries; i++) {
    try {
      var response = UrlFetchApp.fetch(url, {
        muteHttpExceptions: true,
        timeout: 30000 // 30秒
      });

      var responseCode = response.getResponseCode();

      if (responseCode === 200) {
        return JSON.parse(response.getContentText());
      } else if (responseCode === 429) {
        // レート制限超過
        Utilities.sleep(retryDelay * (i + 1));
        continue;
      } else {
        throw new Error('HTTP ' + responseCode + ': ' + response.getContentText());
      }
    } catch (e) {
      if (i === maxRetries - 1) {
        throw e;
      }
      Utilities.sleep(retryDelay * (i + 1));
    }
  }
}
```

### 5. ベストプラクティス

1. **エラーハンドリング:** 常に実装する
2. **リトライ処理:** レート制限や一時的なエラーに対処
3. **タイムアウト設定:** 適切なタイムアウトを設定
4. **ログ記録:** APIリクエストとレスポンスをログに記録
5. **クォータ管理:** 日次クォータを監視
6. **認証情報の保護:** APIキーやトークンをプロパティサービスに保存

---

## Webアプリとしての公開と制限

### 1. Webアプリの要件

スクリプトをWebアプリとして公開するには、以下の要件を満たす必要があります：

#### 1.1 必須関数

- `doGet(e)` 関数（HTTP GETリクエスト用）
- `doPost(e)` 関数（HTTP POSTリクエスト用、オプション）

```javascript
function doGet(e) {
  return HtmlService.createHtmlOutput('Hello World');
}

function doPost(e) {
  var postData = JSON.parse(e.postData.contents);
  return ContentService.createTextOutput(JSON.stringify({
    status: 'success',
    received: postData
  })).setMimeType(ContentService.MimeType.JSON);
}
```

#### 1.2 戻り値

- `HtmlOutput` オブジェクト（HTMLサービス）
- `TextOutput` オブジェクト（コンテンツサービス）

### 2. Webアプリのデプロイ

#### 2.1 デプロイ手順

1. Apps Scriptエディタで「デプロイ」>「新しいデプロイ」をクリック
2. デプロイの種類で「ウェブアプリ」を選択
3. 以下の情報を入力:
   - 説明（任意）
   - 次のユーザーとして実行:
     - **自分（スクリプトの所有者）**: スクリプト所有者の権限で実行
     - **ウェブアプリにアクセスしているユーザー**: アクセスユーザーの権限で実行
   - アクセスできるユーザー:
     - 自分のみ
     - Googleアカウントを持つ全員
     - 全員（匿名ユーザーを含む）

4. 「デプロイ」をクリック
5. ウェブアプリURLが発行される

#### 2.2 実行ユーザーの違い

**「自分（スクリプトの所有者）」として実行:**
- スクリプト所有者の権限で実行
- アクセスユーザーは所有者のデータにアクセス可能
- 認証が不要（匿名アクセスでも実行可能）

**「ウェブアプリにアクセスしているユーザー」として実行:**
- アクセスユーザーの権限で実行
- アクセスユーザーの認証が必要
- ユーザー自身のデータにのみアクセス可能

### 3. リクエストパラメータ

#### 3.1 イベントオブジェクトの構造

| フィールド | 説明 | 例 |
|-----------|------|------|
| `e.queryString` | クエリ文字列全体 | `name=alice&n=1&n=2` |
| `e.parameter` | パラメータオブジェクト（最初の値のみ） | `{"name": "alice", "n": "1"}` |
| `e.parameters` | パラメータオブジェクト（すべての値を配列で） | `{"name": ["alice"], "n": ["1", "2"]}` |
| `e.pathInfo` | `/exec`または`/dev`の後のパス | `/exec/hello` → `hello` |
| `e.contextPath` | 常に空文字列 | `` |
| `e.contentLength` | POSTリクエストの本文の長さ | `332` |
| `e.postData.length` | `e.contentLength`と同じ | `332` |
| `e.postData.type` | POST本文のMIMEタイプ | `text/csv` |
| `e.postData.contents` | POST本文の内容 | `Alice,21` |
| `e.postData.name` | 常に`"postData"` | `postData` |

#### 3.2 使用例

```javascript
function doGet(e) {
  var name = e.parameter.name;
  var age = e.parameter.age;

  return HtmlService.createHtmlOutput(`
    <h1>Hello, ${name}!</h1>
    <p>You are ${age} years old.</p>
  `);
}

// URL例: https://script.google.com/.../exec?name=John&age=30
```

### 4. Webアプリのテスト

#### 4.1 テストデプロイ

1. 「デプロイ」>「テスト導入」をクリック
2. 「ウェブアプリ」を選択
3. ウェブアプリURLをコピー
4. ブラウザでテスト

**テストデプロイの特徴:**
- URLは`/dev`で終わる
- 編集権限を持つユーザーのみアクセス可能
- 最新のコードで実行される

#### 4.2 本番デプロイ

- URLは`/exec`で終わる
- 設定されたアクセス権限に基づいてアクセス可能
- デプロイ時のコードで実行される

### 5. Google Sitesへの埋め込み

1. Webアプリをデプロイ
2. デプロイURLをコピー
3. Google Sitesで「挿入」>「URLを埋め込む」を選択
4. ウェブアプリURLを貼り付け
5. 「追加」をクリック

**注意事項:**
- 埋め込みられたWebアプリは、Google Sitesのドメインで実行される
- サイト閲覧者はWebアプリの承認が必要になる場合がある

### 6. Webアプリの制限

#### 6.1 技術的制限

| 項目 | 制限 |
|------|------|
| 実行時間 | 6分/リクエスト |
| 同時実行数 | 30/ユーザー（1,000/スクリプト） |
| URL Fetchレート制限 | Consumer: 20,000/日<br>Workspace: 100,000/日 |
| サイズ制限 | HTML: 50KB（推奨）<br>レスポンス: 50MB |

#### 6.2 セキュリティ制限

**CSP（Content Security Policy）:**
- インラインJavaScriptは許可されない
- 外部スクリプトの読み込みはホワイトリストに登録されたドメインのみ
- `eval()`関数は使用不可

**回避策:**
```javascript
// HTMLテンプレートを使用
function doGet() {
  var t = HtmlService.createTemplateFromFile('index');
  return t.evaluate();
}

// index.html
<!DOCTYPE html>
<html>
  <head>
    <base target="_top">
  </head>
  <body>
    <h1>Hello World</h1>
    <script>
      // コードは<?!= ... ?>ではなく別ファイルで管理
      google.script.run.myFunction();
    </script>
  </body>
</html>
```

#### 6.3 セッション管理

- Webアプリはステートレス
- セッション情報の保存には`LockService`または`CacheService`を使用

```javascript
function doGet(e) {
  var cache = CacheService.getScriptCache();
  var sessionId = e.parameter.sessionId;

  if (!sessionId) {
    sessionId = Utilities.getUuid();
  }

  cache.put(sessionId, JSON.stringify({
    timestamp: new Date()
  }));

  return HtmlService.createHtmlOutput('Session ID: ' + sessionId);
}
```

### 7. ベストプラクティス

1. **CSPに準拠したコード:** インラインJavaScriptを避ける
2. **エラーハンドリング:** 適切なエラーメッセージを返す
3. **認証:** アクセス制限を適切に設定
4. **バージョン管理:** デプロイバージョンを管理
5. **ログ記録:** `Logger.log`または`console.log`でデバッグ

### 8. REST APIとしての使用

`ContentService`を使用して、JSON形式のレスポンスを返すことができます：

```javascript
function doGet(e) {
  var action = e.parameter.action;
  var response;

  switch (action) {
    case 'list':
      response = {
        status: 'success',
        data: [
          { id: 1, name: 'Item 1' },
          { id: 2, name: 'Item 2' }
        ]
      };
      break;
    case 'get':
      var id = e.parameter.id;
      response = {
        status: 'success',
        data: { id: id, name: 'Item ' + id }
      };
      break;
    default:
      response = {
        status: 'error',
        message: 'Invalid action'
      };
  }

  return ContentService.createTextOutput(JSON.stringify(response))
    .setMimeType(ContentService.MimeType.JSON);
}
```

---

## コンテナバインド vs スタンドアロン

### 1. コンテナバインドスクリプト（Container-bound Scripts）

#### 1.1 定義と特徴

- Google Sheets、Docs、Slides、またはFormsファイルから作成されるスクリプト
- 親ファイルにバインド（紐付け）されている
- Google Driveには別途表示されない
- 親ファイルのライフサイクルに依存する

#### 1.2 作成方法

1. Google Sheets、Docs、Slides、またはFormsを開く
2. 「拡張機能」>「Apps Script」をクリック
3. 新しいApps Scriptプロジェクトが作成される

#### 1.3 メリット

**簡単なデプロイ:**
- ファイル固有の自動化に迅速に対応
- 追加の設定が不要

**直接的なアクセス:**
- `getActiveSheet()`、`getActiveDocument()`等の`getActive*()`メソッドが使用可能
- バインドされているファイルに即座にアクセス

**簡素な構成:**
- ファイル固有の操作に最適
- コードがシンプルになる

#### 1.4 デメリット

**公開の制限:**
- Webアプリとして公開できない
- ライブラリとして公開できない

**再利用性の低さ:**
- 特定のファイルに紐付けられている
- 他のプロジェクトで再利用できない

**スコープの制限:**
- バインドされているファイル内でのみ動作
- 他のファイルにアクセスするには追加の認証が必要

#### 1.5 使用例

```javascript
// コンテナバインドスクリプトの例
function onOpen() {
  var ui = SpreadsheetApp.getUi();
  ui.createMenu('カスタムメニュー')
    .addItem('データを処理', 'processData')
    .addItem('レポート作成', 'createReport')
    .addToUi();
}

function processData() {
  // アクティブなスプレッドシートに直接アクセス
  var sheet = SpreadsheetApp.getActiveSheet();
  var data = sheet.getDataRange().getValues();

  // データ処理
  // ...
}

function createReport() {
  // アクティブなドキュメントに直接アクセス
  var doc = DocumentApp.getActiveDocument();
  var body = body.insertParagraph(0, 'レポート');
}
```

### 2. スタンドアロンスクリプト（Standalone Scripts）

#### 2.1 定義と特徴

- どのGoogle Workspaceファイルにもバインドされていない
- Google Driveに別ファイルとして表示される
- 複数のファイルやプロジェクトにアクセス可能

#### 2.2 作成方法

1. [script.google.com](https://script.google.com)にアクセス
2. 「新しいプロジェクト」をクリック
3. 新しいApps Scriptプロジェクトが作成される

#### 2.3 メリット

**Webアプリとして公開可能:**
- `doGet(e)`、`doPost(e)`関数を使用してWebアプリを作成可能
- 独立したWebアプリケーションとしてデプロイ可能

**ライブラリとして公開可能:**
- 再利用可能なコードライブラリとして作成可能
- 他のプロジェクトからインポート可能

**柔軟性:**
- 複数のドキュメントやプロジェクトにアクセス可能
- より複雑な自動化プロジェクトに適している

**組織化:**
- 複雑な自動化プロジェクトに適した構成
- コードの整理や管理が容易

#### 2.4 デメリット

**明示的な参照が必要:**
- `getActive*()`メソッドが使用不可
- `openById()`、`openByUrl()`等のメソッドで明示的に参照する必要がある

**追加の手間:**
- ドキュメントIDやURLを管理する必要がある
- 認証の設定が必要になる場合がある

#### 2.5 使用例

```javascript
// スタンドアロンスクリプトの例
function processMultipleSpreadsheets() {
  var spreadsheetIds = [
    'SPREADSHEET_ID_1',
    'SPREADSHEET_ID_2',
    'SPREADSHEET_ID_3'
  ];

  spreadsheetIds.forEach(function(id) {
    var ss = SpreadsheetApp.openById(id);
    var sheet = ss.getSheets()[0];
    var data = sheet.getDataRange().getValues();

    // データ処理
    // ...
  });
}

function createReportFromMultipleDocs() {
  var docIds = [
    'DOCUMENT_ID_1',
    'DOCUMENT_ID_2'
  ];

  var reportDoc = DocumentApp.create('統合レポート');
  var body = reportDoc.getBody();

  docIds.forEach(function(id) {
    var doc = DocumentApp.openById(id);
    var text = doc.getBody().getText();
    body.appendParagraph(text);
  });
}

// Webアプリとしての使用
function doGet(e) {
  var action = e.parameter.action;

  switch (action) {
    case 'list':
      return listFiles();
    case 'process':
      return processFiles();
    default:
      return HtmlService.createHtmlOutput('Invalid action');
  }
}
```

### 3. 比較表

| 項目 | コンテナバインド | スタンドアロン |
|------|----------------|---------------|
| **作成元** | Sheets、Docs、Slides、Forms | script.google.com |
| **Google Driveに表示** | いいえ | はい |
| **親ファイルへのバインド** | はい | いいえ |
| **`getActive*()`メソッド** | 使用可能 | 使用不可 |
| **Webアプリとして公開** | 不可 | 可能 |
| **ライブラリとして公開** | 不可 | 可能 |
| **複数ファイルへのアクセス** | 制限あり | 可能 |
| **再利用性** | 低 | 高 |
| **デプロイの手間** | 低 | 中〜高 |
| **用途** | ファイル固有の自動化 | Webアプリ、API、複雑なプロジェクト |

### 4. 選択ガイド

#### 4.1 コンテナバインドスクリプトを使用する場合

- 特定のスプレッドシート、ドキュメント、フォームに対する操作
- カスタム関数の作成
- シンプルな自動化
- プロトタイピングや迅速な開発

**例:**
- スプレッドシートのカスタムメニュー
- 自動レポート生成
- データ検証とフォーマット
- フォーム送信時の自動処理

#### 4.2 スタンドアロンスクリプトを使用する場合

- Webアプリとして公開する場合
- ライブラリとして再利用する場合
- 複数のファイルやプロジェクトにまたがる操作
- 複雑な自動化プロジェクト
- REST APIエンドポイントの作成

**例:**
- Webアプリケーション
- REST API
- ライブラリ
- 複数ファイルの統合処理
- バッチ処理

### 5. 移行方法

#### 5.1 コンテナバインドからスタンドアロンへ

```javascript
// コンテナバインド（変更前）
function processSheet() {
  var sheet = SpreadsheetApp.getActiveSheet();
  // ...
}

// スタンドアロン（変更後）
function processSheet(spreadsheetId) {
  var ss = SpreadsheetApp.openById(spreadsheetId);
  var sheet = ss.getSheets()[0];
  // ...
}
```

#### 5.2 スタンドアロンからコンテナバインドへ

```javascript
// スタンドアロン（変更前）
function processSheet(spreadsheetId) {
  var ss = SpreadsheetApp.openById(spreadsheetId);
  var sheet = ss.getSheets()[0];
  // ...
}

// コンテナバインド（変更後）
function processSheet() {
  var sheet = SpreadsheetApp.getActiveSheet();
  // ...
}
```

### 6. 2025年のベストプラクティス

- **スタンドアロン**は本番アプリケーションに推奨
- **コンテナバインド**はプロトタイピングと迅速な自動化に最適
- 共有や再利用を検討する場合はスタンドアロンを選択

---

## まとめ

### GASの主要な制限まとめ

1. **実行時間:** 6分（カスタム関数は30秒）
2. **1日クォータ:** ConsumerアカウントとGoogle Workspaceアカウントで異なる
3. **メール送信:** Consumerは100宛先/日、Workspaceは1,500宛先/日
4. **トリガー:** 20/ユーザー/スクリプト、総実行時間はConsumer 90分/日、Workspace 6時間/日
5. **URL Fetch:** Consumer 20,000回/日、Workspace 100,000回/日

### 開発者への推奨事項

1. **制限を考慮した設計:** 6分以内で完了する処理に分割
2. **クォータの監視:** 定期的に使用量を確認
3. **適切なトリガーの選択:** シンプルトリガーとインストール可能トリガーの使い分け
4. **外部API連携:** レート制限とIPアドレスの制限に注意
5. **Webアプリ:** CSPに準拠したコードを記述
6. **プロジェクト構成:** 用途に応じてコンテナバインドとスタンドアロンを使い分ける

---

## 参考資料

- [Quotas for Google Services | Apps Script](https://developers.google.com/apps-script/guides/services/quotas)
- [Simple Triggers | Apps Script](https://developers.google.com/apps-script/guides/triggers)
- [Installable Triggers | Apps Script](https://developers.google.com/apps-script/guides/triggers/installable)
- [Web Apps | Apps Script](https://developers.google.com/apps-script/guides/web)
- [Extending Google Sheets | Apps Script](https://developers.google.com/apps-script/guides/sheets)
- [Container-bound Scripts | Apps Script](https://developers.google.com/apps-script/guides/bound)
- [Standalone Scripts | Apps Script](https://developers.google.com/apps-script/guides/standalone)
- [Class UrlFetchApp | Apps Script](https://developers.google.com/apps-script/reference/url-fetch/url-fetch-app)
- [Google Apps Script Quotas & Workarounds (2026) - FolderPal](https://folderpal.io/articles/google-apps-script-quotas-and-workarounds-2026-breaking-limits-on-drive-automation)
