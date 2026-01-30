# Tailwind CSS 入門ガイド

> 非エンジニアのための、まずはこれだけ！個人開発で使えるスタイリング入門

---

## はじめに：Tailwind CSS って何？

「Tailwind（テイルウィンド）CSS」は、**ホームページやウェブアプリの見た目を簡単に整えられるツール**です。

### 🎯 イメージで理解しよう

従来のCSSは「家を建てるとき、すべての家具を一から自分で作る」ようなものです。

Tailwind CSSは「IKEAやニトリで、すでに作られた家具を組み合わせて部屋を作る」ようなものです。

```
従来のCSS:
div.my-button {
  background-color: blue;
  padding: 10px 20px;
  border-radius: 5px;
  color: white;
  text-align: center;
}

Tailwind CSS:
<div class="bg-blue-500 px-5 py-2 rounded text-white text-center">
  ボタン
</div>
```

### 💡 用語解説

| 用語 | 意味 |
|------|------|
| **ユーティリティファースト** | 小さな部品（クラス）を組み合わせてデザインすること |
| **CSSフレームワーク** | スタイリングのルールが決まった道具セット |
| **クラス** | HTMLのタグに付ける「見た目の指示ラベル」 |

---

## 1. インストールと設定

### 1-1. Tailwind CSS とは？

**一言でいうと：コピー＆ペーストでデザインできるツールキット**

- 記述量が減る（コードが短くなる）
- 一貫性のあるデザインができる
- レスポンシブ対応が簡単

### 1-2. Astro プロジェクトへの導入方法

Astro（アストロ）は、高速なウェブサイトを作るためのフレームワークです。

#### ステップ1：インストール

ターミナルで以下を入力：

```bash
npm install -D tailwindcss
npx tailwindcss init
```

> **💡 用語解説**
> - `npm install`：パッケージをダウンロードするコマンド
> - `-D`：開発用としてインストール（本番環境では不要）
> - `npx`：npmパッケージを直接実行するコマンド

#### ステップ2：設定ファイルの作成

`tailwind.config.mjs` というファイルが自動で作られます。これを編集：

```javascript
export default {
  content: ['./src/**/*.{astro,html,js,jsx,md,mdx,svelte,ts,tsx,vue}'],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

> **📝 何をしているの？**
> - `content`：Tailwindを使うファイルの場所を指定
> - `theme`：デザインのカスタマイズ設定
> - `plugins`：追加機能の読み込み

#### ステップ3：CSSファイルの設定

`src/styles/global.css` を作成：

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

#### ステップ4：Astro で読み込み

`src/layouts/Layout.astro` などで：

```astro
---
import '../styles/global.css';
---
<html>
  <head>
    <title>私のサイト</title>
  </head>
  <body>
    <slot />
  </body>
</html>
```

---

## 2. 基本的なユーティリティクラス

### 2-1. 色の指定（text-, bg-, border-）

Tailwind の色指定は「**何の**色」という形式です。

```
text-xxx-yyy  → 文字の色
bg-xxx-yyy    → 背景の色
border-xxx-yyy → 枠線の色
```

#### 基本色の一覧

| 色名 | クラス例 | サンプル |
|------|----------|----------|
| 黒 | `bg-black` | ⬛ |
| 白 | `bg-white` | ⬜ |
| グレー | `bg-gray-500` | ⬛ |
| 赤 | `bg-red-500` | ⬛ |
| 青 | `bg-blue-500` | ⬛ |
| 緑 | `bg-green-500` | ⬛ |

> **💡 数字は何？**
> - `100`：薄い（明るい）
> - `500`：標準
> - `900`：濃い（暗い）
> - 例：`bg-blue-100`（薄い青）〜 `bg-blue-900`（濃い青）

#### 実践例

```html
<!-- 文字が赤い -->
<p class="text-red-500">これは赤い文字です</p>

<!-- 背景が青い -->
<div class="bg-blue-500">背景が青いです</div>

<!-- 枠線が緑 -->
<div class="border-2 border-green-500">緑の枠線</div>
```

### 2-2. 余白と間隔（p-, m-, gap-）

余白は「**どこに**どれくらい」で指定します。

```
p-xxx   → padding（内側の余白）
m-xxx   → margin（外側の余白）
gap-xxx → 要素同士の間隔（flex/grid用）
```

#### 余白の大きさ

| クラス | サイズ | イメージ |
|--------|--------|----------|
| `p-0` | 0px | 余白なし |
| `p-1` | 4px | 小さな余白 |
| `p-2` | 8px | 小さい余白 |
| `p-4` | 16px | 標準的な余白 |
| `p-8` | 32px | 大きな余白 |

#### 方向の指定

| クラス | 方向 |
|--------|------|
| `pt-4` | 上（top） |
| `pr-4` | 右（right） |
| `pb-4` | 下（bottom） |
| `pl-4` | 左（left） |
| `px-4` | 左右（x方向） |
| `py-4` | 上下（y方向） |

> **🧠 覚え方**
> - `px`：横軸（x）＝左右
> - `py`：縦軸（y）＝上下

#### 実践例

```html
<!-- 上下左右に16pxの余白 -->
<div class="p-4">余白のあるボックス</div>

<!-- 上下8px、左右16pxの余白 -->
<div class="py-2 px-4">縦横で違う余白</div>

<!-- 外側の余白 -->
<div class="m-4">外側にスペース</div>
```

### 2-3. フォントサイズ（text-）

文字の大きさは `text-xxx` で指定します。

| クラス | サイズ | 用途 |
|--------|--------|------|
| `text-xs` | 12px | 注釈 |
| `text-sm` | 14px | 小さな文字 |
| `text-base` | 16px | 通常の文字 |
| `text-lg` | 18px | 少し大きい文字 |
| `text-xl` | 20px | 見出し小 |
| `text-2xl` | 24px | 見出し中 |
| `text-4xl` | 36px | 見出し大 |

```html
<h1 class="text-4xl">大きい見出し</h1>
<p class="text-base">通常の文章</p>
<span class="text-sm">小さな注釈</span>
```

### 2-4. 表示・非表示（hidden, block）

| クラス | 挙動 | 使い道 |
|--------|--------|--------|
| `block` | 縦に並ぶ | 通常の表示 |
| `hidden` | 非表示 | モバイルで隠す等 |
| `inline-block` | 横に並ぶ | ボタン等 |

```html
<!-- 通常は表示 -->
<div class="block">見えています</div>

<!-- 隠す -->
<div class="hidden">見えません</div>

<!-- モバイルでは隠す、PCでは表示 -->
<div class="hidden md:block">PCでだけ見える</div>
```

---

## 3. レイアウト（flex, grid）

### 3-1. Flexbox の基本

**Flexbox（フレックスボックス）**：要素を柔軟に並べるレイアウト方法。

イメージ：「ゴム紐で連結された箱」を考えると、伸び縮みして並びます。

#### 基本のクラス

| クラス | 挙動 |
|--------|--------|
| `flex` | Flexbox を有効化 |
| `flex-row` | 横に並べる（デフォルト） |
| `flex-col` | 縦に並べる |
| `justify-center` | 横方向の中央揃え |
| `justify-between` | 両端に配置 |
| `items-center` | 縦方向の中央揃え |
| `items-start` | 上揃え |

#### 実践例：横に並べる

```html
<!-- 横に3つのボックスを並べる -->
<div class="flex gap-4">
  <div class="p-4 bg-blue-500">ボックス1</div>
  <div class="p-4 bg-red-500">ボックス2</div>
  <div class="p-4 bg-green-500">ボックス3</div>
</div>
```

実行結果：
```
[ボックス1] [ボックス2] [ボックス3]
```

#### 実践例：縦に並べる

```html
<!-- 縦に並べる -->
<div class="flex flex-col gap-4">
  <div class="p-4 bg-blue-500">ボックス1</div>
  <div class="p-4 bg-red-500">ボックス2</div>
</div>
```

実行結果：
```
[ボックス1]

[ボックス2]
```

#### 実践例：中央揃え

```html
<!-- 縦横中央に配置 -->
<div class="flex justify-center items-center h-screen">
  <div class="p-8 bg-white rounded shadow">
    真ん中に表示
  </div>
</div>
```

> **💡 用語解説**
> - `h-screen`：画面の高さいっぱい
> - `rounded`：角を丸くする
> - `shadow`：影をつける

### 3-2. Grid の基本

**Grid（グリッド）**：碁盤の目のように要素を配置するレイアウト方法。

イメージ：「Excelの表」のように、行と列で区切って配置します。

#### 基本のクラス

| クラス | 挙動 |
|--------|--------|
| `grid` | Grid を有効化 |
| `grid-cols-2` | 2列にする |
| `grid-cols-3` | 3列にする |
| `grid-cols-4` | 4列にする |
| `gap-4` | 間隔を空ける |

#### 実践例：3列のグリッド

```html
<div class="grid grid-cols-3 gap-4">
  <div class="p-4 bg-blue-500">1</div>
  <div class="p-4 bg-red-500">2</div>
  <div class="p-4 bg-green-500">3</div>
  <div class="p-4 bg-yellow-500">4</div>
  <div class="p-4 bg-purple-500">5</div>
  <div class="p-4 bg-pink-500">6</div>
</div>
```

実行結果：
```
[1] [2] [3]
[4] [5] [6]
```

### 3-3. 具体的なレイアウト例

#### ヘッダー

```html
<header class="flex justify-between items-center p-4 bg-white shadow">
  <h1 class="text-xl font-bold">サイト名</h1>
  <nav class="flex gap-4">
    <a class="text-blue-500">ホーム</a>
    <a class="text-blue-500">About</a>
    <a class="text-blue-500">Contact</a>
  </nav>
</header>
```

> **💡 用語解説**
> - `font-bold`：太字
> - `shadow`：影をつける（立体的に見える）

#### カード

```html
<div class="max-w-sm p-6 bg-white rounded-lg shadow-md">
  <h2 class="text-xl font-bold mb-2">カードタイトル</h2>
  <p class="text-gray-600">カードの説明文です</p>
  <button class="mt-4 px-4 py-2 bg-blue-500 text-white rounded">
    ボタン
  </button>
</div>
```

> **💡 用語解説**
> - `max-w-sm`：最大幅を制限（sm = small）
> - `rounded-lg`：角を大きく丸める
> - `shadow-md`：中くらいの影
> - `mb-2`：margin-bottom（下の余白）

#### グリッドギャラリー

```html
<div class="grid grid-cols-1 md:grid-cols-3 gap-6">
  <div class="bg-white p-4 rounded shadow">アイテム1</div>
  <div class="bg-white p-4 rounded shadow">アイテム2</div>
  <div class="bg-white p-4 rounded shadow">アイテム3</div>
</div>
```

---

## 4. レスポンシブデザイン

### 4-1. ブレークポイントの概念

**ブレークポイント**：デザインが切り替わる画面サイズの境目。

Tailwind のブレークポイント：

| プレフィックス | 最小幅 | 対象デバイス |
|----------------|--------|--------------|
| なし | 0px | すべて（モバイルファースト） |
| `sm:` | 640px | 小型タブレット |
| `md:` | 768px | タブレット |
| `lg:` | 1024px | ノートPC |
| `xl:` | 1280px | デスクトップ |

> **💡 用語解説**
> - **プレフィックス**：クラスの前に付ける接頭辞（例：`md:text-xl`）
> - **最小幅**：これ以上の幅で適用される

### 4-2. モバイルファーストの考え方

「まずモバイルで考えて、大きい画面ではデザインを追加していく」考え方です。

```
✅ モバイルファースト（推奨）
<div class="text-sm md:text-base lg:text-lg">
  → モバイル：小さい文字
  → タブレット：普通の文字
  → PC：大きい文字
</div>

❌ 逆の手順（避ける）
<div class="text-lg md:text-base">
  → 大きい画面から考えると、書くのが大変
</div>
```

### 4-3. レスポンシブ対応の実例

#### 文字サイズの変更

```html
<h1 class="text-2xl md:text-4xl lg:text-6xl">
  レスポンシブ見出し
</h1>
```

| 画面サイズ | 文字サイズ |
|------------|------------|
| モバイル | 24px |
| タブレット | 36px |
| PC | 60px |

#### カラムの変更

```html
<!-- モバイル：1列 → タブレット：2列 → PC：3列 -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  <div>アイテム1</div>
  <div>アイテム2</div>
  <div>アイテム3</div>
</div>
```

#### 表示・非表示の切り替え

```html
<!-- モバイルでは非表示、PCでは表示 -->
<div class="hidden md:block">
  PCでだけ見えるコンテンツ
</div>

<!-- モバイルでは表示、PCでは非表示 -->
<div class="block md:hidden">
  モバイルでだけ見えるコンテンツ
</div>
```

#### メニューの切り替え

```html
<nav class="flex flex-col md:flex-row gap-4">
  <a>ホーム</a>
  <a>About</a>
  <a>Contact</a>
</nav>
```

| 画面サイズ | 表示 |
|------------|------|
| モバイル | 縦に並ぶ |
| PC | 横に並ぶ |

---

## 5. カスタム設定

### 5-1. カラーパレットの追加

`tailwind.config.mjs` で独自の色を追加できます。

```javascript
export default {
  theme: {
    extend: {
      colors: {
        'brand': '#3b82f6',        // ブルー
        'brand-dark': '#1e40af',   // ダークブルー
        'accent': '#f59e0b',       // オレンジ
      }
    }
  }
}
```

使用例：

```html
<button class="bg-brand text-white hover:bg-brand-dark">
  ボタン
</button>
```

> **💡 用語解説**
> - `hover:`：マウスが乗った時のスタイル
> - `extend`：既存の設定に追加

### 5-2. カスタムクラスの作成（@apply）

よく使う組み合わせをクラスとしてまとめられます。

`src/styles/global.css` に記述：

```css
@layer components {
  .btn-primary {
    @apply px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-700;
  }

  .card {
    @apply p-6 bg-white rounded-lg shadow-md;
  }
}
```

使用例：

```html
<!-- クラス一つで済む -->
<button class="btn-primary">送信</button>

<div class="card">カードの内容</div>
```

> **💡 用語解説**
> - `@layer components`：コンポーネント用のレイヤー
> - `@apply`：Tailwindのクラスを適用する命令

### 5-3. テーマの設定

フォントや間隔などの基本設定を変更できます。

```javascript
export default {
  theme: {
    extend: {
      fontFamily: {
        sans: ['Noto Sans JP', 'sans-serif'],
      },
      spacing: {
        '18': '4.5rem',  // 独自の間隔を追加
      }
    }
  }
}
```

---

## 6. よくあるミスと対処法

| ミス | 原因 | 対処法 |
|------|------|--------|
| スタイルが効かない | `tailwind.config.mjs`の`content`設定ミス | ファイルパスを確認 |
| レスポンシブが動かない | `md:`等のプレフィックスの前が空いていない | スペースを入れる |
| スペースが効かない | Flexbox/Grid の中で`gap`未使用 | `gap-4`等を使う |
| カスタム色が使えない | 設定ファイルを更新していない | `npm run dev`を再起動 |

### 💡 トラブルシューティング

1. **開発サーバーを再起動する**
   ```bash
   # Ctrl+C で停止してから
   npm run dev
   ```

2. **ブラウザのキャッシュをクリア**
   - Chrome: `Cmd + Shift + R`

3. **クラス名のスペルチェック**
   - `text-red500` ❌
   - `text-red-500` ✅（ハイフンが必要）

---

## 7. 用語集

| 用語 | 読み方 | 意味 |
|------|--------|------|
| ユーティリティクラス | - | 単一の目的を持つ小さなクラス |
| Flexbox | フレックスボックス | 要素を柔軟に配置するレイアウト方法 |
| Grid | グリッド | 行と列で配置するレイアウト方法 |
| ブレークポイント | - | レスポンシブデザインの切り替え点 |
| モバイルファースト | - | モバイルからデザインを始める手法 |
| プレフィックス | - | クラス名の前に付ける接頭辞（`md:`等） |
| パディング | - | 要素の内側の余白 |
| マージン | - | 要素の外側の余白 |
| レスポンシブ | - | 画面サイズに合わせて変化すること |
| フレームワーク | - | 開発を支援する道具セット |

---

## まとめチェックリスト

作成が完了したらチェックしてみましょう！

- [ ] Tailwind CSS がインストールできた
- [ ] 設定ファイル（`tailwind.config.mjs`）を作成した
- [ ] 基本的な色、余白、フォントサイズを使える
- [ ] Flexbox で横・縦に並べられる
- [ ] Grid で表のように配置できる
- [ ] レスポンシブ対応（`md:`等）が使える
- [ ] カスタム色やクラスを作成できる

---

## 次のステップ

- [ ] 公式ドキュメントを見る：https://tailwindcss.com/docs
- [ ] コンポーネント集を参考にする：https://tailwindui.com/
- [ ] 実際にサイトを作ってみる

---

**Happy Coding! 🎨**
