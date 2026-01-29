# Gmail・Sheets連携リサーチ

> **研究日時**: 2026-01-29
> **研究者**: 足軽3号
> **目的**: GmailService と SheetsService の連携手法を徹底的に解明する

## 目次

1. [GmailService](#1-gmailservice)
2. [SheetsService / SpreadsheetApp](#2-sheetsservice--spreadsheetapp)
3. [GmailApp vs MailApp](#3-gmailapp-vs-mailapp)
4. [バッチ処理と大量データ処理](#4-バッチ処理と大量データ処理)
5. [実践的なコード例](#5-実践的なコード例)

---

## 1. GmailService

### 1.1 概要

GmailServiceは、Gmailアカウントのメール、スレッド、下書き、ラベル、添付ファイルにアクセス・操作できるサービスである。

### 1.2 主要クラス

| クラス | 説明 |
|--------|------|
| `GmailApp` | Gmailのスレッド、メッセージ、ラベルにアクセスするエントリポイント |
| `GmailMessage` | メールメッセージを表す |
| `GmailThread` | スレッド（メッセージの集合）を表す |
| `GmailDraft` | 下書きメッセージを表す |
| `GmailLabel` | ユーザー作成のラベルを表す |
| `GmailAttachment` | メールの添付ファイルを表す |

### 1.3 GmailApp 主要メソッド

#### メールの取得・検索

```javascript
// スレッド取得
var threads = GmailApp.getInboxThreads();  // 受信トレイの全スレッド
var threads = GmailApp.search('subject:"重要"');  // 検索

// メッセージ取得
var messages = threads[0].getMessages();
var message = GmailApp.getMessageById(messageId);

// 特定フォルダ
var priorityThreads = GmailApp.getPriorityInboxThreads();
var starredThreads = GmailApp.getStarredThreads();
var spamThreads = GmailApp.getSpamThreads();
var trashThreads = GmailApp.getTrashThreads();
```

#### メールの送信

```javascript
// 基本的な送信
GmailApp.sendEmail('recipient@example.com', '件名', '本文');

// オプション付き送信
GmailApp.sendEmail('recipient@example.com', '件名', '本文', {
  cc: 'cc@example.com',
  bcc: 'bcc@example.com',
  name: '送信者名',
  replyTo: 'reply@example.com',
  attachments: [blob],
  htmlBody: '<b>HTML本文</b>',
  inlineImages: {
    imageId: blob
  }
});
```

#### ラベル操作

```javascript
// ラベルの作成・取得
var label = GmailApp.createLabel('プロジェクト');
var label = GmailApp.getUserLabelByName('プロジェクト');
var labels = GmailApp.getUserLabels();

// スレッドにラベルを追加
thread.addLabel(label);

// スレッドからラベルを削除
thread.removeLabel(label);
```

#### メールの状態変更

```javascript
// 既読/未読
GmailApp.markMessageRead(message);
GmailApp.markMessageUnread(message);
GmailApp.markThreadsRead(threads);
GmailApp.markThreadsUnread(threads);

// 重要マーク
GmailApp.markThreadImportant(thread);
GmailApp.markThreadsUnimportant(threads);

// スター
GmailApp.starMessage(message);
GmailApp.unstarMessages(messages);
```

#### メールの移動

```javascript
// アーカイブ、迷惑メール、ゴミ箱へ移動
GmailApp.moveThreadToArchive(thread);
GmailApp.moveThreadToSpam(thread);
GmailApp.moveThreadToTrash(thread);
GmailApp.moveThreadsToInbox(threads);
```

### 1.4 GmailMessage 主要メソッド

```javascript
// メッセージ情報の取得
var subject = message.getSubject();      // 件名
var body = message.getBody();            // HTML本文
var plainBody = message.getPlainBody();  // プレーンテキスト本文
var from = message.getFrom();            // 送信者
var to = message.getTo();                // 宛先
var cc = message.getCc();                // CC
var date = message.getDate();            // 日時
var id = message.getId();                // メッセージID

// 添付ファイル
var attachments = message.getAttachments();
for (var i = 0; i < attachments.length; i++) {
  var blob = attachments[i].getAsBlob();
  var fileName = attachments[i].getName();
}

// 返信・転送
message.reply('返信本文');
message.replyAll('全員に返信');
message.forward('forward@example.com');

// 下書き作成
var draft = message.createDraftReply('下書き本文');
```

---

## 2. SheetsService / SpreadsheetApp

### 2.1 概要

SpreadsheetAppは、Googleスプレッドシートにアクセス・操作するためのサービスである。シート、範囲（Range）、セルなどを操作できる。

### 2.2 主要クラス

| クラス | 説明 |
|--------|------|
| `SpreadsheetApp` | スプレッドシート操作のエントリポイント |
| `Spreadsheet` | スプレッドシート全体を表す |
| `Sheet` | シート（ワークシート）を表す |
| `Range` | セルまたはセルの範囲を表す |

### 2.3 Rangeオブジェクトの基本

#### Rangeの取得方法

```javascript
var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('シート1');

// A1記法で取得
var range1 = sheet.getRange('A1:B10');

// 行列インデックスで取得（1-based）
var range2 = sheet.getRange(1, 1, 10, 2);  // 行1, 列1, 高さ10, 幅2 = A1:B10

// 名前付き範囲
var range3 = SpreadsheetApp.getActive().getRangeByName('データ範囲');

// データ範囲（A1から連続するデータがある範囲）
var range4 = sheet.getDataRange();

// アクティブな範囲
var range5 = SpreadsheetApp.getActive().getActiveRange();
```

#### データの読み書き

```javascript
// 値の取得（2次元配列で返される）
var values = range.getValues();
// values[0][0] は A1 の値
// values[2][1] は C3 の値

// 値の設定（2次元配列を渡す必要あり）
var data = [
  ['名前', '年齢', '得点'],
  ['太郎', 25, 85],
  ['花子', 28, 92]
];
range.setValues(data);

// 単一セルの取得・設定
var value = range.getValue();
range.setValue(100);

// 数式の取得・設定
var formulas = range.getFormulas();
range.setFormula('=SUM(A1:A10)');
range.setFormulas([
  ['=SUM(A1:A10)', '=AVERAGE(B1:B10)'],
  ['=MAX(C1:C10)', '=MIN(D1:D10)']
]);
```

### 2.4 Rangeの主要メソッド

#### データ操作

```javascript
// クリア
range.clear();           // 内容と書式をクリア
range.clearContent();    // 内容のみクリア
range.clearFormat();     // 書式のみクリア

// コピー・貼り付け
range.copyTo(destination);                    // 値と書式をコピー
range.copyTo(destination, {contentsOnly:true}); // 値のみコピー
range.copyValuesToRange(sheet, column, columnEnd, row, rowEnd);

// 移動
range.moveTo(target);  // カット＆ペースト
```

#### フォーマット

```javascript
// 背景色
range.setBackground('#ff0000');
range.setBackgrounds([['#ff0000', '#00ff00'], ['#0000ff', '#ffff00']]);

// フォント
range.setFontColor('#ffffff');
range.setFontFamily('Arial');
range.setFontSize(12);
range.setFontWeight('bold');
range.setFontStyle('italic');

// 配置
range.setHorizontalAlignment('center');
range.setVerticalAlignment('middle');

// 罫線
range.setBorder(true, true, true, true, '#000000', SpreadsheetApp.BorderStyle.SOLID);
```

#### 行列操作

```javascript
// 情報取得
var numRows = range.getNumRows();
var numColumns = range.getNumColumns();
var rowIndex = range.getRowIndex();
var columnIndex = range.getColumn();

// セルの取得
var cell = range.getCell(row, column);
```

---

## 3. GmailApp vs MailApp

### 3.1 違いの要点

| 項目 | GmailApp | MailApp |
|------|----------|---------|
| **目的** | 送信 + Gmail操作 | 送信のみ |
| **Gmailアクセス** | あり（スレッド、メッセージ、ラベル） | なし |
| **スコープ** | 広い（Gmailへの完全アクセス） | 狭い（送信のみ） |
| **エイリアス使用** | 可能 | 不可 |
| **Gmail必須** | 是 | 否（Google Workspaceのみ可） |

### 3.2 使い分け

#### GmailApp を使う場合

```javascript
// Gmailのデータを読み取る必要がある
var threads = GmailApp.getInboxThreads();
var labels = GmailApp.getUserLabels();

// エイリアスで送信する必要がある
GmailApp.sendEmail(to, subject, body, {
  from: GmailApp.getAliases()[0]
});

// 下書きを作成する
var draft = GmailApp.createDraft(to, subject, body);

// メールを検索する
var threads = GmailApp.search('is:unread label:"重要"');
```

#### MailApp を使う場合

```javascript
// シンプルにメールを送るだけ
MailApp.sendEmail('recipient@example.com', '件名', '本文');

// スコープを最小限にしたい（アドオン配布時など）
// ユーザーに「Gmailへのアクセス」を許可させたくない場合
```

### 3.3 非推奨（Deprecated）について

**MailApp は非推奨ではない**。古いサービス（Contacts serviceなど）は非推奨化されたが、MailAppは現在も有効である。

---

## 4. バッチ処理と大量データ処理

### 4.1 ベストプラクティス

#### 原則

1. **API呼び出しを最小限にする** - 1回の呼び出しでまとめて処理
2. **ループ内でAPI呼び出しをしない** - 重大なパフォーマンス低下
3. **メモリ内で処理してから一括書き込み** - 読み取り→処理→書き込み

#### 悪い例

```javascript
// ❌ 非常に遅い（1000回のAPI呼び出し）
for (var i = 1; i <= 1000; i++) {
  sheet.getRange(i, 1).setValue(i * 2);
}
```

#### 良い例

```javascript
// ✅ 高速（1回の読み取り + 1回の書き込み）
var range = sheet.getRange(1, 1, 1000, 1);
var values = range.getValues();

for (var i = 0; i < values.length; i++) {
  values[i][0] = (i + 1) * 2;
}

range.setValues(values);
```

### 4.2 大量データ処理のテクニック

#### データ範囲を一括取得

```javascript
var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
var dataRange = sheet.getDataRange();  // A1からデータがある範囲全て
var values = dataRange.getValues();

// 処理
for (var i = 0; i < values.length; i++) {
  var row = values[i];
  // 行ごとの処理
}
```

#### バッチサイズで分割処理

```javascript
var MAX_BATCH_SIZE = 1000;
var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
var lastRow = sheet.getLastRow();

for (var i = 1; i <= lastRow; i += MAX_BATCH_SIZE) {
  var endRow = Math.min(i + MAX_BATCH_SIZE - 1, lastRow);
  var range = sheet.getRange(i, 1, endRow - i + 1, 5);  // 5列分
  var values = range.getValues();

  // 処理
  var processed = processBatch(values);

  // 書き込み
  range.setValues(processed);
}
```

### 4.3 パフォーマンス比較

| 処理内容 | 悪い実装 | 良い実装 | 効果 |
|----------|----------|----------|------|
| 1000セル書き込み | セル単位のsetValue | 一括setValues | 約100倍高速 |
| 10,000行読み取り | 行単位のgetValue | getDataRange | 約50倍高速 |
| 条件付き書式 | セル単位設定 | 範囲単位設定 | 約10倍高速 |

---

## 5. 実践的なコード例

### 5.1 Gmailから特定条件のメールを取得してSheetsに書き込む

```javascript
function exportGmailToSheets() {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('メール');
  if (!sheet) {
    sheet = SpreadsheetApp.getActiveSpreadsheet().insertSheet('メール');
  }

  // Gmailを検索
  var threads = GmailApp.search('subject:"プロジェクト" is:unread');

  // データ収集
  var data = [];
  for (var i = 0; i < threads.length; i++) {
    var messages = threads[i].getMessages();
    for (var j = 0; j < messages.length; j++) {
      var msg = messages[j];
      data.push([
        msg.getDate(),
        msg.getFrom(),
        msg.getSubject(),
        msg.getPlainBody().substring(0, 100) + '...'
      ]);
    }
  }

  // シートに書き込み
  if (data.length > 0) {
    sheet.getRange(2, 1, data.length, 4).setValues(data);
    // 既読にする
    GmailApp.markThreadsRead(threads);
  }
}
```

### 5.2 Sheetsのデータを元にメールを一括送信

```javascript
function sendBulkEmails() {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('送信リスト');
  var data = sheet.getDataRange().getValues();

  // ヘッダーをスキップ
  for (var i = 1; i < data.length; i++) {
    var row = data[i];
    var to = row[0];      // A列: メールアドレス
    var name = row[1];    // B列: 名前
    var status = row[4];  // E列: ステータス

    if (status !== '送信済み') {
      GmailApp.sendEmail(to, 'こんにちは' + name + 'さん', '本文...');

      // ステータスを更新
      sheet.getRange(i + 1, 5).setValue('送信済み');
    }
  }
}
```

### 5.3 Sheetsのデータを加工して別シートに転記

```javascript
function transformAndCopy() {
  var sourceSheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('元データ');
  var targetSheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('集計');

  // 元データを一括取得
  var sourceData = sourceSheet.getDataRange().getValues();

  // データ変換
  var transformed = sourceData.map(function(row) {
    return [
      row[0],           // 名前
      row[1] * 1.1,     // 数値を1.1倍
      new Date(row[2]), // 日付変換
      row[3] ? '〇' : '×'  // ブール値を文字列に
    ];
  });

  // 転記
  targetSheet.getRange(1, 1, transformed.length, transformed[0].length)
    .setValues(transformed);
}
```

### 5.4 添付ファイルをGoogle Driveに保存

```javascript
function saveAttachments() {
  var folder = DriveApp.getFolderById('フォルダID');
  var threads = GmailApp.search('has:attachment');

  for (var i = 0; i < threads.length; i++) {
    var messages = threads[i].getMessages();
    for (var j = 0; j < messages.length; j++) {
      var attachments = messages[j].getAttachments();
      for (var k = 0; k < attachments.length; k++) {
        var blob = attachments[k].getAsBlob();
        folder.createFile(blob);
      }
    }
  }
}
```

---

## 参考情報

### 公式ドキュメント

- [Gmail Service | Apps Script | Google for Developers](https://developers.google.com/apps-script/reference/gmail)
- [Class Range | Apps Script | Google for Developers](https://developers.google.com/apps-script/reference/spreadsheet/range)
- [Best Practices | Apps Script](https://developers.google.com/apps-script/guides/support/best-practices)

### 関連リソース

- [Reading from and writing to a Range in Google Sheets](https://spreadsheet.dev/reading-from-writing-to-range-in-google-sheets-using-apps-script)

---

*以上、足軽3によるリサーチ報告でござる。*
