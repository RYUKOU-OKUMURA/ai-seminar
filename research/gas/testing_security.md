# Google Apps Script - テスト・CI/CDとセキュリティ

> **Research Date**: 2026-01-29
> **Researcher**: Ashigaru8
> **Task ID**: gas_research_008

---

## 目次

1. [テスト手法](#テスト手法)
2. [CI/CD連携](#cicd連携)
3. [セキュリティベストプラクティス](#セキュリティベストプラクティス)
4. [まとめ](#まとめ)

---

## テスト手法

### テストフレームワーク

Google Apps Script には以下のテストフレームワークが利用可能である：

| フレームワーク | 説明 | ライブラリID |
|---------------|------|-------------|
| **GSUnit** | GAS専用のテストフレームワーク | `1VXHbMTCarTMcx6H_EHHhaVay4ptnl2I6FjTU8wl0my4ALmM7nLGuZ0WJ` |
| **QUnit for GAS** | QUnitのGASポート版 | [GitHub](https://github.com/google/qunit) |
| **gas-fakes** | ローカル開発・テスト用モックライブラリ | [gas-fakes](https://github.com/LoopThrough/gas-fakes) |

### GSUnit の使用例

```javascript
// GSUnitを使ったテスト例
function testCalculateTotal() {
  // Arrange
  const values = [100, 200, 300];

  // Act
  const result = calculateTotal(values);

  // Assert
  assertEquals(600, result, "Total should be 600");
}

function testCalculateTotalWithEmptyArray() {
  const result = calculateTotal([]);
  assertEquals(0, result, "Empty array should return 0");
}
```

### QUnit for GAS の使用例

```javascript
// QUnitを使用したテスト例
QUnit.test("GmailService test", function(assert) {
  const emails = GmailApp.getInboxThreads();

  assert.ok(emails.length >= 0, "Should retrieve inbox threads");
  assert.ok(typeof emails === "object", "Should return an array");
});
```

### 単体テストの書き方

#### 基本的なテスト構造

```javascript
/**
 * 単体テストの基本パターン
 */
function testFeature() {
  // 1. 準備 (Arrange)
  const input = "test data";

  // 2. 実行 (Act)
  const result = processInput(input);

  // 3. 検証 (Assert)
  if (result !== "expected") {
    throw new Error("Test failed: expected 'expected', got '" + result + "'");
  }
}
```

#### テスト対象の関数例

```javascript
/**
 * 文字列をキャピタライズする関数
 */
function capitalize(str) {
  if (!str || typeof str !== "string") {
    return "";
  }
  return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();
}

/**
 * テストケース
 */
function testCapitalize() {
  // 正常系
  assertEquals("Hello", capitalize("hello"));
  assertEquals("World", capitalize("WORLD"));

  // 異常系
  assertEquals("", capitalize(null));
  assertEquals("", capitalize(""));
  assertEquals("", capitalize(123));
}

/**
 * カスタムアサーション関数
 */
function assertEquals(expected, actual, message) {
  if (expected !== actual) {
    const errorMsg = message
      ? message + " (expected: " + expected + ", actual: " + actual + ")"
      : "Expected " + expected + " but got " + actual;
    throw new Error(errorMsg);
  }
}
```

### 外部からのテスト実行

#### 1. Execution API を使用したテスト実行

Apps Script Execution API を使用すると、外部から関数を実行できる：

```bash
# gcloud コマンドでの実行例
gcloud scripts call SCRIPT_ID \
  --data='{"functionName":"testFeature"}'
```

#### 2. clasp によるテスト実行

```bash
# ローカルから関数を実行
clasp run testFeature

# 非同期実行
clasp run testFeature --async
```

#### 3. Webアプリとしてテストを実行

```javascript
// テストランナー用のWebアプリエンドポイント
function doGet() {
  const results = runAllTests();
  return ContentService.createTextOutput(JSON.stringify(results))
    .setMimeType(ContentService.MimeType.JSON);
}

function runAllTests() {
  const tests = [
    testCapitalize,
    testCalculateTotal,
    testGmailService
  ];

  const results = {
    total: tests.length,
    passed: 0,
    failed: 0,
    details: []
  };

  tests.forEach(test => {
    try {
      test();
      results.passed++;
      results.details.push({ name: test.name, status: "PASSED" });
    } catch (e) {
      results.failed++;
      results.details.push({
        name: test.name,
        status: "FAILED",
        error: e.message
      });
    }
  });

  return results;
}
```

---

## CI/CD連携

### Clasp を使ったデプロイ自動化

#### 基本的なデプロイフロー

```bash
# 1. プル（最新の状態を取得）
clasp pull

# 2. プッシュ（変更をアップロード）
clasp push

# 3. バージョン作成
clasp create-version "Release v1.0.0"

# 4. デプロイ
clasp deploy
```

### GitHub Actions との統合

#### 基本的なワークフロー設定

```yaml
name: Deploy to Google Apps Script

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Install clasp
        run: npm install -g @google/clasp

      - name: Setup .clasprc.json
        run: |
          echo "$CLASPRC_JSON" > .clasprc.json
        env:
          CLASPRC_JSON: ${{ secrets.CLASPRC_JSON }}

      - name: Push to Apps Script
        run: |
          clasp push --force

      - name: Create deployment
        run: |
          clasp deploy --description "Deploy from GitHub Actions"
```

#### TypeScript + Rollup を使用する場合

```yaml
name: Build and Deploy

on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build

      - name: Install clasp
        run: npm install -g @google/clasp

      - name: Deploy
        env:
          CLASPRC_JSON: ${{ secrets.CLASPRC_JSON }}
        run: |
          echo "$CLASPRC_JSON" > .clasprc.json
          clasp push --force
          clasp deploy --description "Release ${{ github.sha }}"
```

### Clasp Action を使用した簡略化

```yaml
name: Deploy with Clasp Action

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Apps Script
        uses: naman1909/clasp-action@v2
        with:
          CLASPRC_KEYS: ${{ secrets.CLASPRC_JSON }}
          PROJECT_ID: ${{ secrets.PROJECT_ID }}
          SCRIPT_ID: ${{ secrets.SCRIPT_ID }}
          COMMAND: "push --force"
```

### CI/CD 認証の設定

#### .clasprc.json の取得と保存

```bash
# 1. clasp でログイン
clasp login

# 2. .clasprc.json の内容をコピー
cat ~/.clasprc.json

# 3. GitHub Secrets に登録
# Repository > Settings > Secrets > New secret
# Name: CLASPRC_JSON
# Value: {cat の出力}
```

### 継続的インテグレーション例

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npm run type-check

      - name: Unit tests
        run: npm test

  deploy-production:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Deploy to production
        env:
          CLASPRC_JSON: ${{ secrets.CLASPRC_JSON }}
        run: |
          npm install -g @google/clasp
          echo "$CLASPRC_JSON" > .clasprc.json
          clasp push --force
          clasp deploy --description "Production release ${{ github.sha }}"
```

---

## セキュリティベストプラクティス

### OAuth2 認証

#### スコープの最小化原則

Apps Script では OAuth2 スコープを使用して権限を管理する：

```javascript
/**
 * appsscript.json で明示的にスコープを指定
 */
{
  "oauthScopes": [
    "https://www.googleapis.com/auth/spreadsheets.readonly",
    "https://www.googleapis.com/auth/gmail.send"
  ],
  "runtimeVersion": "V8",
  "exceptionLogging": "STACKDRIVER"
}
```

#### 推奨されるスコープ設定

| 必要な機能 | 推奨スコープ | 避けるべきスコープ |
|-----------|-------------|------------------|
| スプレッドシートの読み取り | `spreadsheets.readonly` | `spreadsheets` |
| Gmail 送信のみ | `gmail.send` | `gmail` |
| ドライブの特定ファイル操作 | `drive.file` | `drive` |

#### OAuth2 フローの実装

```javascript
/**
 * OAuth2 サービスを使用した外部API連携
 */
function getOAuth2Service() {
  return OAuth2.createService('MyService')
    .setAuthorizationBaseUrl('https://example.com/oauth/authorize')
    .setTokenUrl('https://example.com/oauth/token')
    .setClientId(PropertiesService.getScriptProperties().getProperty('CLIENT_ID'))
    .setClientSecret(PropertiesService.getScriptProperties().getProperty('CLIENT_SECRET'))
    .setCallbackFunction('authCallback')
    .setPropertyStore(PropertiesService.getUserProperties());
}

/**
 * OAuth2 コールバック処理
 */
function authCallback(request) {
  const service = getOAuth2Service();
  const isAuthorized = service.handleCallback(request);
  return HtmlService.createHtmlOutput(isAuthorized ? 'Success!' : 'Denied');
}
```

### Lock Service による排他制御

#### Lock Service の基本

```javascript
/**
 * Lock Service を使用した排他制御
 */
function criticalSectionWithLock() {
  const lock = LockService.getScriptLock();

  try {
    // ロックの取得（最大30秒待機）
    lock.waitLock(30000);

    // クリティカルセクションの処理
    const data = readSharedData();
    const processed = processData(data);
    writeSharedData(processed);

  } catch (e) {
    console.error('Lock acquisition failed: ' + e.message);
    throw e;
  } finally {
    // ロックの解放
    lock.releaseLock();
  }
}
```

#### 排他制御の使用シナリオ

| シナリオ | 説明 | ロックタイプ |
|----------|------|-------------|
| スプレッドシート更新 | 同時書き込み防止 | `getScriptLock()` |
| ユーザー単位の操作 | ユーザー間の競合防止 | `getUserLock()` |
| ドキュメント単位 | ドキュメントごとの排他 | `getDocumentLock()` |

### センシティブデータの管理

#### PropertiesService での暗号化

**注意**: PropertiesService はデフォルトでは暗号化されない。センシティブデータを扱う場合は暗号化が必要。

```javascript
/**
 * 暗号化ユーティリティ
 */
function encryptData(data, key) {
  return Utilities.base64Encode(
    Utilities.computeDigest(Utilities.DigestAlgorithm.SHA_256, data + key)
  );
}

function decryptData(encryptedData, key) {
  // 実際の実装では適切な暗号化ライブラリを使用
  const decoded = Utilities.base64Decode(encryptedData);
  return Utilities.newBlob(decoded).getDataAsString();
}

/**
 * センシティブデータの安全な保存
 */
function saveApiKey(apiKey) {
  const scriptProps = PropertiesService.getScriptProperties();
  const encryptionKey = scriptProps.getProperty('ENCRYPTION_KEY');
  const encrypted = encryptData(apiKey, encryptionKey);
  scriptProps.setProperty('ENCRYPTED_API_KEY', encrypted);
}

function getApiKey() {
  const scriptProps = PropertiesService.getScriptProperties();
  const encryptionKey = scriptProps.getProperty('ENCRYPTION_KEY');
  const encrypted = scriptProps.getProperty('ENCRYPTED_API_KEY');
  return decryptData(encrypted, encryptionKey);
}
```

#### Secret Service ライブラリの使用

より堅牢なシークレット管理には、以下のライブラリが利用可能：

- [secret-service](https://github.com/dataful-tech/secret-service) - GAS用のシークレット管理ライブラリ

### Webアプリ公開時のセキュリティ

#### アクセス権限の設定

```javascript
/**
 * Webアプリとして公開する際のセキュリティ考慮
 */
function doGet(e) {
  // ユーザー認証の確認
  const user = Session.getActiveUser();
  if (user.getEmail() === '') {
    return ContentService.createTextOutput('Unauthorized')
      .setMimeType(ContentService.MimeType.TEXT);
  }

  // 許可されたドメインからのアクセスのみ許可
  const allowedDomains = ['example.com', 'trusted-domain.com'];
  const referer = e.parameter.referer;

  if (referer && !isDomainAllowed(referer, allowedDomains)) {
    return ContentService.createTextOutput('Forbidden')
      .setMimeType(ContentService.MimeType.TEXT);
  }

  // メイン処理
  return processRequest(e);
}

function isDomainAllowed(url, allowedDomains) {
  const hostname = new URL(url).hostname;
  return allowedDomains.some(domain => hostname.endsWith(domain));
}
```

### APIキー管理のベストプラクティス

#### 環境変数の使用

```javascript
/**
 * スクリプトプロパティからAPIキーを取得
 */
function getApiConfig() {
  const props = PropertiesService.getScriptProperties();

  return {
    apiKey: props.getProperty('API_KEY'),
    endpoint: props.getProperty('API_ENDPOINT'),
    timeout: parseInt(props.getProperty('API_TIMEOUT') || '30000')
  };
}

/**
 * シークレットのローテーション対応
 */
function refreshApiKey() {
  const newKey = fetchNewKeyFromSecureStore();
  const props = PropertiesService.getScriptProperties();
  props.setProperty('API_KEY', newKey);
  props.setProperty('API_KEY_ROTATED_AT', new Date().toISOString());
}
```

### セキュリティチェックリスト

| 項目 | チェック内容 |
|------|-------------|
| スコープ | 最小権限の原則を適用しているか |
| 認証 | OAuth2 を適切に実装しているか |
| 暗号化 | センシティブデータを暗号化しているか |
| 排他制御 | 競合を防ぐために Lock Service を使用しているか |
| 入力検証 | ユーザー入力を適切に検証しているか |
| エラーハンドリング | センシティブ情報をエラーメッセージに含めていないか |

---

## まとめ

### テスト手法

- **GSUnit / QUnit**: GAS で使用可能なテストフレームワーク
- **単体テスト**: Arrange-Act-Assert パターンで実装
- **外部実行**: Execution API、clasp、Webアプリエンドポイントを使用

### CI/CD連携

- **Clasp**: コマンドラインからのデプロイ自動化
- **GitHub Actions**: 継続的インテグレーションとデプロイ
- **認証**: .clasprc.json をシークレットとして管理

### セキュリティ

- **OAuth2**: スコープを最小化して権限を管理
- **Lock Service**: 排他制御で競合を防止
- **暗号化**: PropertiesService のデータは暗号化が必要
- **Webアプリ**: アクセス制限と入力検証を実装

---

## 参考リンク

### テスト
- [GSUnit - GitHub](https://github.com/LaughDonor/GSUnit)
- [QUnit for Google Apps Script](https://github.com/google/qunit)
- [gas-fakes - Local Development](https://github.com/LoopThrough/gas-fakes)
- [How to Modularize and Test Google Apps Scripts](https://dev.to/davepar/how-to-modularize-and-test-google-apps-scripts-4ig2)

### CI/CD
- [Clasp Action - GitHub Marketplace](https://github.com/marketplace/actions/clasp-action)
- [Automate GAS Deployments with GitHub Actions](https://dev.to/gkukurin/how-to-automate-google-apps-script-deployments-with-github-actions-36dk)
- [GAS in VS Code with Clasp - The Apps Script Lab](https://theappsscriptlab.com/google-apps-script-in-vs-code-with-clasp-local-dev-git-and-ci-cd-the-right-way/)
- [CI/CD Guide - GitHub Actions Integration](https://en.zhgchg.li/posts/zrealm-dev/ci-cd-guide-integrate-google-apps-script-web-app-with-github-actions-for-free-packaging-platform-4273e57e7148/)

### セキュリティ
- [Apps Script Best Practices](https://developers.google.com/apps-script/guides/support/best-practices)
- [Google OAuth2 Best Practices](https://developers.google.com/identity/protocols/oauth2/resources/best-practices)
- [Apps Script Authorization Scopes](https://developers.google.com/apps-script/concepts/scopes)
- [GAS Security Best Practices for Developers](https://levelup.gitconnected.com/google-apps-script-security-best-practices-for-developers-fff9207d847b)
- [Properties Service Documentation](https://developers.google.com/apps-script/guides/properties)
- [Secret Service Library](https://github.com/dataful-tech/secret-service)
- [Using Named Locks with GAS](https://ramblings.mcpher.com/gassnippets2/using-named-locks-with-google-apps-scripts/)

---

*このリサーチドキュメントは Google Apps Script のテスト手法・CI/CDとセキュリティベストプラクティスに関する包括的な概要を提供するものである。*
