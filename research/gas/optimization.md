# GAS 最適化・ベストプラクティス

> Google Apps Scriptのパフォーマンス最適化と制限回避に関する包括的ガイド
>
> **作成日**: 2026-01-29
> **リサーチ元**: Google公式ドキュメント及び最新のベストプラクティス

---

## 目次

1. [実行時間制限（6分）の回避テクニック](#1-実行時間制限6分の回避テクニック)
2. [大量データ処理の最適化](#2-大量データ処理の最適化)
3. [Googleサービスのコール数削減](#3-googleサービスのコール数削減)
4. [PropertiesServiceによるキャッシング](#4-propertiesserviceによるキャッシング)
5. [トリガーの分割と継続処理パターン](#5-トリガーの分割と継続処理パターン)
6. [メモリ使用量の最適化](#6-メモリ使用量の最適化)
7. [非同期処理のパターンと限界](#7-非同期処理のパターンと限界)
8. [パフォーマンス計測方法](#8-パフォーマンス計測方法)
9. [クォータと制限一覧](#9-クォータと制限一覧)

---

## 1. 実行時間制限（6分）の回避テクニック

### 1.1 制限の概要

| 項目 | 制限時間 |
|------|----------|
| スクリプト実行時間 | 6分 / 実行 |
| カスタム関数実行時間 | 30秒 / 実行 |
| Workspaceアドオン実行時間 | 30秒 / 実行 |

**参考**: [Quotas for Google Services | Apps Script](https://developers.google.com/apps-script/guides/services/quotas)

### 1.2 トリガーによる自己再実行パターン

処理の進捗を保存し、時間経過後に自分自身を再実行する「不死鳥関数」パターン。

```javascript
// 不死鳥関数の実装例
function processLargeData() {
  const PROPS = PropertiesService.getScriptProperties();
  let startIndex = parseInt(PROPS.getProperty('startIndex') || '0');
  const BATCH_SIZE = 1000; // 1回の処理件数

  // データ取得
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const lastRow = sheet.getLastRow();
  const data = sheet.getRange(startIndex + 1, 1, lastRow - startIndex, 1).getValues();

  // 処理実行（5分以内で完了する量に制限）
  const endTime = new Date().getTime() + 5 * 60 * 1000; // 現在時刻 + 5分
  let processedCount = 0;

  for (let i = 0; i < data.length; i++) {
    if (new Date().getTime() >= endTime) {
      // 時間制限に近づいたら中断
      PROPS.setProperty('startIndex', String(startIndex + processedCount));
      ScriptApp.newTrigger('processLargeData')
               .timeBased()
               .after(1000) // 1秒後に再実行
               .create();
      return;
    }

    // ここで実際の処理
    processRow(data[i]);
    processedCount++;
  }

  // 完了時はプロパティを削除
  PROPS.deleteProperty('startIndex');
}

function processRow(row) {
  // 個別の行処理ロジック
}
```

**参考**: [GASの「6分の壁」を再帰処理で突破する方法](https://cp-agency.co.jp/jidoukanosusume/case_studies/gas-recursion-6minute-wall/)

### 1.3 再帰処理による分割

処理を複数回の実行に分割して実行する方法。

```javascript
function recursiveProcessor(continuationToken) {
  const options = {
    pageSize: 100,
    pageToken: continuationToken
  };

  const result = processDataBatch(options);

  if (result.nextPageToken) {
    // 次のページがあればトリガーで再実行
    ScriptApp.newTrigger('recursiveProcessor')
             .timeBased()
             .after(5000)
             .create()
             .setUniqueId(result.nextPageToken);
  }
}
```

### 1.4 HTML側での継続呼び出し

クライアント側から継続的にGASを呼び出す手法。

```javascript
// クライアント側（HTML）
function continueProcessing(token) {
  google.script.run
    .withSuccessHandler(function(result) {
      if (result.continue) {
        setTimeout(() => continueProcessing(result.token), 1000);
      } else {
        alert('処理完了');
      }
    })
    .withFailureHandler(function(error) {
      console.error(error);
    })
    .processBatch(token);
}

// サーバー側
function processBatch(token) {
  const batchSize = 100;
  const result = processItems(token, batchSize);

  const remaining = getRemainingCount();
  if (remaining > 0) {
    return {
      continue: true,
      token: result.nextToken
    };
  }
  return { continue: false };
}
```

---

## 2. 大量データ処理の最適化

### 2.1 バッチ操作（Batch Operations）

**重要**: 交互の読み書きは極めて遅い。一括読み込み→処理→一括書き込みが鉄則。

#### 悪い例（70秒かかる）

```javascript
// DO NOT USE - 遅いコード
var cell = sheet.getRange('a1');
for (var y = 0; y < 100; y++) {
  for (var x = 0; x < 100; x++) {
    var c = getColorFromCoordinates(x, y);
    cell.offset(y, x).setBackgroundColor(c); // 10,000回のAPI呼び出し
  }
  SpreadsheetApp.flush();
}
```

#### 良い例（1秒で完了）

```javascript
// RECOMMENDED - 高速コード
var cell = sheet.getRange('a1');
var colors = new Array(100);
for (var y = 0; y < 100; y++) {
  colors[y] = new Array(100);
  for (var x = 0; x < 100; x++) {
    colors[y][x] = getColorFromCoordinates(x, y);
  }
}
sheet.getRange(1, 1, 100, 100).setBackgrounds(colors); // 1回のAPI呼び出し
```

**効果**: 70秒 → 1秒（70倍高速化）

### 2.2 スプレッドシート操作のベストプラクティス

```javascript
// データの一括読み込み
const data = sheet.getDataRange().getValues();

// メモリ上で全処理を実行
const processed = data.map(row => {
  return processRowInMemory(row);
});

// 結果を一括書き込み
sheet.getRange(1, 1, processed.length, processed[0].length)
     .setValues(processed);
```

**参考**: [Best Practices | Apps Script](https://developers.google.com/apps-script/guides/support/best-practices)

### 2.3 バッチ処理のベンチマーク

| 操作方法 | 100x100セル処理時間 |
|----------|-------------------|
| セル単位の書き込み | ~70秒 |
| バッチ書き込み | ~1秒 |

---

## 3. Googleサービスのコール数削減

### 3.1 原則：JavaScript処理を優先

```javascript
// 悪い例 - 多数のAPI呼び出し
for (var i = 0; i < 1000; i++) {
  var value = SpreadsheetApp.getActiveSpreadsheet()
                               .getActiveSheet()
                               .getRange(i, 1)
                               .getValue();
  processValue(value);
}

// 良い例 - 1回のAPI呼び出し
const values = SpreadsheetApp.getActiveSpreadsheet()
                              .getActiveSheet()
                              .getDataRange()
                              .getValues();
for (var i = 0; i < values.length; i++) {
  processValue(values[i][0]);
}
```

**ポイント**: Google Apps Script内のJavaScript操作は、外部サービス呼び出しより圧倒的に高速。

### 3.2 組み込みキャッシュの活用

GASにはルックアヘッドキャッシュとライトバックキャッシュが組み込まれている。

```javascript
// 組み込みキャッシュを最大限活用
// 連続した読み取りはキャッシュから提供される
const range = sheet.getRange("A1:Z1000");
const values = range.getValues(); // キャッシュに格納される
// 再度同じ範囲を読んでもキャッシュから取得（高速）
```

### 3.3 外部サービス呼び出しの最小化

```javascript
// CacheServiceで外部APIの結果をキャッシュ
function fetchExternalData() {
  const cache = CacheService.getScriptCache();
  const cached = cache.get("external-data");

  if (cached != null) {
    return JSON.parse(cached);
  }

  // 初回のみ外部APIを呼び出す
  const response = UrlFetchApp.fetch("https://api.example.com/data");
  const data = response.getContentText();

  // 25分間キャッシュ
  cache.put("external-data", data, 1500);
  return JSON.parse(data);
}
```

---

## 4. PropertiesServiceによるキャッシング

### 4.1 PropertiesServiceの種類

| 種類 | スコープ | 主な用途 |
|------|---------|----------|
| ScriptProperties | スクリプト全体 | 全ユーザー共通の設定 |
| DocumentProperties | ドキュメント | ドキュメント固有の設定 |
| UserProperties | ユーザー | ユーザー固有の設定 |

### 4.2 使用例

```javascript
// スクリプトプロパティの使用
function saveProgress(index) {
  const props = PropertiesService.getScriptProperties();
  props.setProperty('lastProcessedIndex', String(index));
}

function loadProgress() {
  const props = PropertiesService.getScriptProperties();
  return parseInt(props.getProperty('lastProcessedIndex') || '0');
}

// オブジェクトの保存（JSONシリアライズ）
function saveState(state) {
  const props = PropertiesService.getScriptProperties();
  props.setProperty('appState', JSON.stringify(state));
}

function loadState() {
  const props = PropertiesService.getScriptProperties();
  const stateJson = props.getProperty('appState');
  return stateJson ? JSON.parse(stateJson) : null;
}
```

### 4.3 制限事項

| 項目 | Consumer | Workspace |
|------|----------|-----------|
| 読み書き回数 | 50,000 / 日 | 500,000 / 日 |
| 値のサイズ | 9 KB / 値 | 9 KB / 値 |
| 総ストレージ | 500 KB | 500 KB |

### 4.4 CacheServiceとの使い分け

| サービス | 有効期限 | 用途 |
|----------|----------|------|
| CacheService | 最大6時間 | 一時的なキャッシュ |
| PropertiesService | 半永久的 | 進捗保存、設定値 |

---

## 5. トリガーの分割と継続処理パターン

### 5.1 トリガーの種類と制限

| トリガータイプ | 最大数 | 実行時間制限 |
|----------------|--------|--------------|
| 時間駆動型トリガー | 20 / ユーザー / スクリプト | 6分 / 実行 |
| 手動トリガー | - | 6分 / 実行 |
| シンプルトリガー | 自動生成 | 30秒 |

### 5.2 時間駆動型トリガーの作成

```javascript
// 分単位のトリガー
ScriptApp.newTrigger('myFunction')
         .timeBased()
         .everyMinutes(10)
         .create();

// 特定時刻のトリガー
ScriptApp.newTrigger('dailyJob')
         .timeBased()
         .atHour(9)
         .everyDays(1)
         .create();

// 継続処理用のトリガー（1回のみ）
ScriptApp.newTrigger('continueProcessing')
         .timeBased()
         .after(60000) // 1分後
         .create();
```

### 5.3 継続処理パターン実装

```javascript
function continueProcessing() {
  const props = PropertiesService.getScriptProperties();
  const state = JSON.parse(props.getProperty('processState') || '{}');

  // 現在のトリガーを削除
  const triggers = ScriptApp.getProjectTriggers();
  triggers.forEach(t => {
    if (t.getHandlerFunction() === 'continueProcessing') {
      ScriptApp.deleteTrigger(t);
    }
  });

  // 処理実行
  const result = processBatch(state);

  if (!result.completed) {
    // 次の状態を保存
    props.setProperty('processState', JSON.stringify(result.nextState));

    // 次のトリガーを設定
    ScriptApp.newTrigger('continueProcessing')
             .timeBased()
             .after(1000)
             .create();
  } else {
    // 完了時は状態をクリア
    props.deleteProperty('processState');
  }
}
```

### 5.4 トリガー実行時間のクォータ

| アカウント種別 | 1日あたりの総実行時間 |
|----------------|---------------------|
| Consumer | 90分 / 日 |
| Google Workspace | 6時間 / 日 |

---

## 6. メモリ使用量の最適化

### 6.1 メモリエラーの対処法

```
Exception: Service using too much memory
```

このエラーが発生した場合の対策：

1. **データの分割処理**: 一度に処理するデータ量を減らす
2. **不要なオブジェクトの解放**: 大きな配列やオブジェクトを`null`に設定
3. **ストリーミング処理**: 全データをメモリに載せずに処理

```javascript
// メモリ効率の良い処理
function processInBatches() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const totalRows = sheet.getLastRow();
  const batchSize = 1000;

  for (let startRow = 1; startRow <= totalRows; startRow += batchSize) {
    const endRow = Math.min(startRow + batchSize - 1, totalRows);
    const data = sheet.getRange(startRow, 1, endRow - startRow + 1, 10).getValues();

    // 処理実行
    processBatch(data);

    // メモリ解放（次のイテレーションのために）
    data = null;
  }
}
```

**参考**: ["Exception: Service using too much memory" の対処法](https://error-daizenn.hatenablog.com/entry/2025/01/14/163217)

### 6.2 メモリ節約テクニック

1. **大きな配列を避ける**: 必要最小限のデータのみを保持
2. **文字列結合の最適化**: `+=` ではなく `push().join()` を使用
3. **正規表現の再利用**: コンパイル済み正規表現を使い回す

```javascript
// 文字列結合の最適化
// 悪い例
let result = "";
for (let i = 0; i < 10000; i++) {
  result += items[i]; // メモリ再確保が多発
}

// 良い例
const parts = [];
for (let i = 0; i < 10000; i++) {
  parts.push(items[i]);
}
const result = parts.join('');
```

---

## 7. 非同期処理のパターンと限界

### 7.1 GASの非同期処理モデル

**重要**: Google Apps Scriptはシングルスレッドで実行され、JavaScriptの`async/await`や`Promise`は期待通りに動作しない。

### 7.2 非同期処理の限界

```javascript
// これは期待通りに動作しない
async function fetchData() {
  const response = await UrlFetchApp.fetch(url); // awaitは無効
  return response;
}
```

### 7.3 トリガーを使用した擬似非同期処理

```javascript
// タスクをキューに追加
function enqueueTask(taskData) {
  const props = PropertiesService.getScriptProperties();
  let queue = JSON.parse(props.getProperty('taskQueue') || '[]');
  queue.push({
    data: taskData,
    timestamp: new Date().toISOString()
  });
  props.setProperty('taskQueue', JSON.stringify(queue));

  // 処理トリガーがなければ作成
  if (!hasProcessingTrigger()) {
    ScriptApp.newTrigger('processQueue')
             .timeBased()
             .after(1000)
             .create();
  }
}

// キューの処理
function processQueue() {
  const props = PropertiesService.getScriptProperties();
  let queue = JSON.parse(props.getProperty('taskQueue') || '[]');

  if (queue.length === 0) {
    return; // タスクなし
  }

  // 先頭タスクを処理
  const task = queue.shift();
  processTask(task.data);

  // 残りタスクを保存
  props.setProperty('taskQueue', JSON.stringify(queue));

  // 残りタスクがあれば次のトリガーを設定
  if (queue.length > 0) {
    ScriptApp.newTrigger('processQueue')
             .timeBased()
             .after(1000)
             .create();
  }
}
```

### 7.4 並列実行の制限

| 項目 | 制限 |
|------|------|
| 同時実行数（ユーザーあたり） | 30 |
| 同時実行数（スクリプトあたり） | 1,000 |

---

## 8. パフォーマンス計測方法

### 8.1 実行時間の計測

```javascript
function measurePerformance() {
  const startTime = new Date().getTime();

  // 測定対象の処理
  targetFunction();

  const endTime = new Date().getTime();
  const executionTime = endTime - startTime;

  console.log(`実行時間: ${executionTime}ms`);
  return executionTime;
}
```

### 8.2 Google Cloud Platformでの詳細計測

```javascript
// Stackdriver Loggingを使用した詳細計測
function logMetrics() {
  const startTime = Date.now();

  // 処理実行
  const result = processData();

  const duration = Date.now() - startTime;

  // メトリクスログ
  console.log(JSON.stringify({
    metric: 'execution_time',
    value: duration,
    function: 'processData'
  }));
}
```

### 8.3 キャッシュヒット率の計測

```javascript
function getCachedData(key) {
  const cache = CacheService.getScriptCache();
  const props = PropertiesService.getScriptProperties();

  // ヒット率計測
  let hits = parseInt(props.getProperty('cache_hits') || '0');
  let misses = parseInt(props.getProperty('cache_misses') || '0');

  const cached = cache.get(key);
  if (cached != null) {
    hits++;
    props.setProperty('cache_hits', String(hits));
    return JSON.parse(cached);
  }

  misses++;
  props.setProperty('cache_misses', String(misses));

  // キャッシュにない場合の処理
  const data = fetchFreshData(key);
  cache.put(key, JSON.stringify(data), 1500);

  // ヒット率をログ
  const total = hits + misses;
  const hitRate = (hits / total * 100).toFixed(2);
  console.log(`キャッシュヒット率: ${hitRate}% (${hits}/${total})`);

  return data;
}
```

**目標**: キャッシュヒット率75%以上

### 8.4 API呼び出し回数の追跡

```javascript
// API呼び出し回数のカウンター
function trackApiCall(serviceName) {
  const props = PropertiesService.getScriptProperties();
  const key = `api_calls_${serviceName}`;
  let count = parseInt(props.getProperty(key) || '0');
  count++;
  props.setProperty(key, String(count));
}

// 使用例
function getSpreadsheetData() {
  trackApiCall('spreadsheet');
  return SpreadsheetApp.getActiveSpreadsheet().getDataRange().getValues();
}
```

---

## 9. クォータと制限一覧

### 9.1 1日あたりのクォータ

| 機能 | Consumer | Google Workspace |
|------|----------|------------------|
| カレンダーイベント作成 | 5,000 | 10,000 |
| コンタクト作成 | 1,000 | 2,000 |
| ドキュメント作成 | 250 | 1,500 |
| ファイル変換 | 2,000 | 4,000 |
| **メール送信先（1日）** | **100** | **1,500** |
| メール読み書き | 20,000 | 50,000 |
| プロパティ読み書き | 50,000 | 500,000 |
| **トリガー総実行時間** | **90分/日** | **6時間/日** |
| URL Fetch呼び出し | 20,000 | 100,000 |

### 9.2 技術的制限

| 項目 | 制限 |
|------|------|
| **スクリプト実行時間** | **6分 / 実行** |
| カスタム関数実行時間 | 30秒 / 実行 |
| 同時実行数（ユーザー） | 30 |
| メール添付ファイル | 250 / メッセージ |
| メール本文サイズ | 200 KB (Consumer), 400 KB (Workspace) |
| プロパティ値サイズ | 9 KB / 値 |
| プロパティ総ストレージ | 500 KB |
| トリガー数 | 20 / ユーザー / スクリプト |
| URL Fetchレスポンスサイズ | 50 MB |

**参考**: [Quotas for Google Services | Apps Script](https://developers.google.com/apps-script/guides/services/quotas)

### 9.3 例外メッセージ一覧

| メッセージ | 原因 |
|-----------|------|
| `Limit exceeded: Email Attachments Per Message.` | メール添付ファイル数超過 |
| `Service invoked too many times: Calendar.` | サービス呼び出し回数超過 |
| `Service invoked too many times in a short time: Calendar.` | 短時間での呼び出し過多 |
| `Service using too much computer time for one day.` | 1日の実行時間超過 |
| `Script invoked too many times per second.` | 1秒あたりの呼び出し過多 |
| `There are too many scripts running simultaneously.` | 同時実行数超過 |

---

## まとめ：最適化チェックリスト

- [ ] バッチ操作を使用（一括読み書き）
- [ ] 外部サービス呼び出しを最小化
- [ ] CacheServiceを活用
- [ ] 6分制限を考慮した分割処理
- [ ] API呼び出し回数を追跡
- [ ] メモリ使用量を最適化
- [ ] パフォーマンス計測を実施

---

## 参考リソース

- [Best Practices | Apps Script | Google Developers](https://developers.google.com/apps-script/guides/support/best-practices)
- [Quotas for Google Services | Apps Script | Google Developers](https://developers.google.com/apps-script/guides/services/quotas)
- [GASの「6分の壁」を再帰処理で突破する方法](https://cp-agency.co.jp/jidoukanosusume/case_studies/gas-recursion-6minute-wall/)
- [【GAS】Google App Scriptのタイムアウトエラーを回避する方法](https://rightcode.co.jp/blogs/52115)
- [GASスプレッドシート読み書きを高速化](https://tonari-it.com/gas-spreadsheet-speedup/)
- [GASの実行を高速化するテクニック](https://note.com/gicloud/n/nf2aad6f99152)
- ["Exception: Service using too much memory" の対処法](https://error-daizenn.hatenablog.com/entry/2025/01/14/163217)
