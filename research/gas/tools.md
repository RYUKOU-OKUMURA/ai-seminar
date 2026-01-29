# Google Apps Script - Clasp & V8 Runtime

> 作成日: 2026-01-29
> 最終更新: 2026-01-29

## 目次

1. [Clasp とは](#clasp-とは)
2. [Clasp のインストールとセットアップ](#clasp-のインストールとセットアップ)
3. [Clasp コマンド一覧](#clasp-コマンド一覧)
4. [ローカル開発環境の構築](#ローカル開発環境の構築)
5. [TypeScript との統合](#typescript-との統合)
6. [ESLint、Prettier 等のツールチェーン](#eslintprettier-等のツールチェーン)
7. [.clasp.json の設定](#claspjson-の設定)
8. [V8 ランタイムの詳細](#v8-ランタイムの詳細)
9. [旧 Rhino ランタイムとの互換性問題](#旧-rhino-ランタイムとの互換性問題)

---

## Clasp とは

**Clasp**（Command Line Apps Script Projects）は、Google Apps Script プロジェクトをローカル環境で開発するための CLI ツールである。

### 主な機能

- **ローカル開発**: ソースコードをバージョン管理し、好きなエディタで開発可能
- **デプロイ管理**: バージョン管理とデプロイの作成・更新
- **フォルダ構造**: script.google.com のフラット構造をローカルのフォルダ構造に自動変換
- **関数実行**: コマンドラインから Apps Script 関数を実行

> 公式リポジトリ: [google/clasp - GitHub](https://github.com/google/clasp)

---

## Clasp のインストールとセットアップ

### インストール

```bash
npm install -g @google/clasp
```

### 事前要件

- **Node.js**: v22.0.0 以上

```bash
node -v  # バージョン確認
```

### Google Apps Script API の有効化

1. [https://script.google.com/home/usersettings](https://script.google.com/home/usersettings) にアクセス
2. 「Google Apps Script API」を有効にする

### ログイン

```bash
# 基本ログイン
clasp login

# ローカルサーバーを使用しない場合
clasp login --no-localhost

# カスタム認証情報を使用
clasp login --creds client_secret.json

# 特定ポートを指定
clasp login --redirect-port 37473

# 複数ユーザー管理
clasp login --user testaccount
```

---

## Clasp コマンド一覧

### 基本コマンド

| コマンド | 説明 |
|----------|------|
| `clasp login` | ユーザーログイン |
| `clasp logout` | ログアウト |
| `clasp create-script` | 新規スクリプト作成 |
| `clasp clone-script` | スクリプトをクローン |
| `clasp delete-script` | スクリプト削除 |
| `clasp pull` | リモートからファイル取得 |
| `clasp push` | ローカルファイルをアップロード |
| `clasp show-file-status` | ファイル状態確認 |
| `clasp open-script` | スクリプトエディタを開く |
| `clasp list-deployments` | デプロイ一覧表示 |
| `clasp create-deployment` | デプロイ作成 |
| `clasp delete-deployment` | デプロイ削除 |
| `clasp create-version` | バージョン作成 |
| `clasp list-versions` | バージョン一覧表示 |
| `clasp list-scripts` | スクリプト一覧表示 |

### 高度なコマンド（Project ID 必要）

| コマンド | 説明 |
|----------|------|
| `clasp tail-logs` | StackDriver ログ表示 |
| `clasp list-apis` | 利用可能な API 一覧 |
| `clasp enable-api` | API を有効化 |
| `clasp disable-api` | API を無効化 |
| `clasp run-function` | 関数を実行 |

### コマンド詳細

#### create-script

```bash
# スタンドアロン
clasp create-script --type standalone

# Docs アドオン
clasp create-script --type docs

# Sheets アドオン
clasp create-script --type sheets

# Web アプリ
clasp create-script --type webapp

# API
clasp create-script --type api

# タイトル付き
clasp create-script --title "My Script"

# ルートディレクトリ指定
clasp create-script --rootDir ./dist

# 親ファイル指定
clasp create-script --parentId "1D_Gxyv*****************************NXO7o"
```

#### clone-script

```bash
# Script ID でクローン
clasp clone-script "15ImUCpyi1Jsd8yF8Z6wey_7cw793CymWTLxOqwMka3P1CzE5hQun6qiC"

# URL でクローン
clasp clone-script "https://script.google.com/d/15ImUCpyi1Jsd8yF8Z6wey_7cw793CymWTLxOqwMka3P1CzE5hQun6qiC/edit"

# バージョン指定
clasp clone-script "ScriptID" --versionNumber 23

# ルートディレクトリ指定
clasp clone-script "ScriptID" --rootDir ./src
```

#### push

```bash
# プッシュ
clasp push

# 強制上書き
clasp push -f

# ファイル監視（変更時自動プッシュ）
clasp push --watch
```

#### pull

```bash
# プル
clasp pull

# バージョン指定
clasp pull --versionNumber 23

# 未使用ファイル削除
clasp pull --deleteUnusedFiles
```

---

## ローカル開発環境の構築

### ディレクトリ構造例

```
my-gas-project/
├── .clasp.json          # Clasp 設定ファイル
├── .claspignore         # 除外ファイル設定
├── appsscript.json      # Apps Script マニフェスト
├── src/
│   ├── main.ts
│   ├── utils.ts
│   └── Code.gs
├── test/
│   └── main.test.ts
├── tsconfig.json
├── package.json
├── .eslintrc.cjs
└── .prettierrc
```

### .claspignore の設定

`.gitignore` と同様に、`clasp push` で除外するファイルを設定する。

```
# 全てのファイルを無視
**/**

# 除外するファイルを指定
!build/main.js
!appsscript.json
```

デフォルトの `.claspignore`:

```
# ignore all files…
**/**

# except the extensions…
!appsscript.json
!**/*.gs
!**/*.js
!**/*.ts
!**/*.html

# ignore even valid files if in…
.git/**
node_modules/**
```

---

## TypeScript との統合

### Clasp 3.x の重要な変更

**Clasp 3.x は TypeScript のトランスパイルをサポートしなくなった。**

代わりに、以下の方法が推奨される：

1. **Bundler（Rollup 等）を使用**
2. **事前に TypeScript を変換**

### 推奨テンプレートプロジェクト

- [apps-script-engine-template](https://github.com/WildH0g/apps-script-engine-template)
- [clasp-typescript-template](https://github.com/tomoyanakano/clasp-typescript-template)
- [aside](https://github.com/google/aside)
- [apps-script-typescript-rollup-starter](https://github.com/sqrrrl/apps-script-typescript-rollup-starter)

### tsconfig.json 設定例

```json
{
  "compilerOptions": {
    "target": "ES2019",
    "module": "ESNext",
    "lib": ["ES2019"],
    "outDir": "./build",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "types": ["@types/google-apps-script"]
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

---

## ESLint、Prettier 等のツールチェーン

### ESLint 設定

```javascript
// .eslintrc.cjs
module.exports = {
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 2020,
    sourceType: 'module',
  },
  plugins: ['@typescript-eslint'],
  rules: {
    '@typescript-eslint/no-explicit-any': 'warn',
  },
};
```

### Prettier 設定

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2
}
```

### package.json 設定例

```json
{
  "name": "my-gas-project",
  "version": "1.0.0",
  "scripts": {
    "build": "rollup -c",
    "lint": "eslint src/**/*.ts",
    "format": "prettier --write src/**/*.ts",
    "push": "npm run build && clasp push"
  },
  "devDependencies": {
    "@typescript-eslint/eslint-plugin": "^7.0.0",
    "@typescript-eslint/parser": "^7.0.0",
    "eslint": "^8.0.0",
    "prettier": "^3.0.0",
    "rollup": "^4.0.0",
    "typescript": "^5.0.0"
  }
}
```

---

## .clasp.json の設定

### 設定項目

```json
{
  "scriptId": "",                    // 必須: Apps Script プロジェクト ID
  "rootDir": "./build",              // 任意: プロジェクトファイルのディレクトリ
  "projectId": "project-id-xxx",     // 任意: GCP プロジェクト ID
  "scriptExtensions": [".js", ".gs"], // 任意: スクリプトファイルの拡張子
  "htmlExtensions": [".html"],       // 任意: HTML ファイルの拡張子
  "filePushOrder": ["file1.ts", "file2.ts"] // 任意: プッシュ順序
}
```

### scriptId の取得方法

1. [script.google.com](https://script.google.com) を開く
2. ファイル > プロジェクトのプロパティ > スクリプト ID

### projectId の設定方法

1. `clasp open-script` でプロジェクトを開く
2. リソース > Cloud Platform プロジェクト...
3. プロジェクト ID を指定

---

## V8 ランタイムの詳細

### V8 ランタイムとは

Google Apps Script は 2020 年に Rhino エンジンから **V8 エンジン**（Chrome や Node.js と同じエンジン）に移行した。

### ⚠️ 重要：2026 年 1 月 31 日期限

**Rhino ランタイムは 2026 年 1 月 31 日をもって廃止される。**

全ての Apps Script プロジェクトは、この日付までに V8 ランタイムへ移行する必要がある。

> 参照: [Migrate your Apps Script projects to V8 runtime before Jan 31, 2026](https://discuss.google.dev/t/migrate-your-apps-script-projects-to-v8-runtime-before-jan-31-2026/260396)

### サポートされる ES6+ 機能

V8 ランタイムでは以下のモダン JavaScript 機能がサポートされている：

| 機能 | サポート |
|------|----------|
| アロー関数 | ✅ |
| `const` / `let` | ✅ |
| テンプレートリテラル | ✅ |
| 分割代入 | ✅ |
| デフォルトパラメータ | ✅ |
| スプレッド演算子 | ✅ |
| レストパラメータ | ✅ |
| クラス | ✅ |
| async/await | ✅ |
| Promise | ✅ |
| Map/Set | ✅ |
| シンボル | ✅ |

### ⚠️ サポートされない機能

| 機能 | ステータス |
|------|----------|
| **ES6 モジュール（import/export）** | ❌ 未サポート |
| ファイルベースのモジュールエクスポート | ❌ 未サポート |

> 重要: V8 ランタイムでも **ES6 モジュールは使用できない**。従来の Apps Script ファイルインクルードモデルを使用する必要がある。

### V8 ランタイムへの移行

#### マニフェストで有効化

```json
{
  "timeZone": "Asia/Tokyo",
  "dependencies": {},
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8"
}
```

#### 移行ガイド

公式の移行ガイド: [Migrate scripts to the V8 runtime](https://developers.google.com/apps-script/guides/v8-runtime/migration)

---

## 旧 Rhino ランタイムとの互換性問題

### 主な互換性問題

| 問題 | 説明 | 対応 |
|------|------|------|
| `Date.parse()` | ISO 8601 形式の厳格な解析 | `new Date()` を使用 |
| `String.prototype.match()` | グローバル正規表現の挙動変更 | ループで `exec()` 使用 |
| `for...in` | プロパティ列挙順序の変更 | `Object.keys()` 使用 |
| `Array.prototype.sort()` | 安定ソートへの変更 | 依存コードを確認 |

### 移行チェックリスト

- [ ] `runtimeVersion: "V8"` をマニフェストに追加
- [ ] 非推奨メソッドを置き換え
- [ ] テストで動作確認
- [ ] エラーログを確認

### 互換性のあるコード例

```typescript
// Rhino と V8 両方で動作するコード
function formatDate(date: Date): string {
  return Utilities.formatDate(
    date,
    Session.getScriptTimeZone(),
    'yyyy-MM-dd HH:mm:ss'
  );
}

// 正規表現の安全な使用
function findMatches(text: string, pattern: string): string[] {
  const regex = new RegExp(pattern, 'g');
  const matches: string[] = [];
  let match;
  while ((match = regex.exec(text)) !== null) {
    matches.push(match[1]);
  }
  return matches;
}
```

---

## まとめ

### Clasp を使う利点

1. **バージョン管理**: Git でのコード管理が可能
2. **モダン開発**: ESLint、Prettier、TypeScript 統合
3. **CI/CD**: 自動デプロイ pipeline 構築可能
4. **チーム開発**: 複数人での並行開発が容易

### V8 ランタイムへの移行

- **期限**: 2026 年 1 月 31 日までに必須
- **ES6+ 機能**: ほぼサポート（ただし ES6 モジュールは除く）
- **移行方法**: `runtimeVersion: "V8"` をマニフェストに追加

### 推奨開発環境

```json
{
  "runtimeVersion": "V8",
  "timeZone": "Asia/Tokyo",
  "exceptionLogging": "STACKDRIVER",
  "executionApi": {
    "access": "ANYONE"
  }
}
```

---

## 参考リンク

- [Clasp 公式 GitHub](https://github.com/google/clasp)
- [V8 Runtime Overview | Apps Script](https://developers.google.com/apps-script/guides/v8-runtime)
- [Migrate scripts to the V8 runtime](https://developers.google.com/apps-script/guides/v8-runtime/migration)
- [JavaScript modules - MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
