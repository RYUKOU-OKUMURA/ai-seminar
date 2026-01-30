# Astro（アストロ）入門ガイド
## 非エンジニアのためのWebサイト作成フレームワーク

> **対象者**: これから個人開発に挑戦したい非エンジニアの方
> **難易度**: 初級（プログラミング経験がなくても大丈夫！）

---

## はじめに：このガイドについて

このガイドは、プログラミングに詳しくない方でも「自分のWebサイトを作りたい」と思ったら、すぐに始められるように作りました。

専門用語には必ず解説をつけています。分からない言葉があったら、「用語集」セクションを確認してください。

---

# 1. Astroの基本概念と特徴

## 1-1. Astroとは何か？

**Astro（アストロ）**は、Webサイトを作るための「道具箱」です。

料理で例えると：
- レストランの厨房 = フレームワーク（Astro）
- レシピ = コード
- 料理人 = あなた

Astroを使うと、複雑な手順を書かなくても、美味しいWebサイト（料理）を作ることができます。

### 静的サイトジェネレーターとは？

「静的」という言葉は難しく聞こえますが、要するに「**あらかじめ作っておいたページをそのまま表示する**」という意味です。

- **静的サイト**: ページが完成した状態で保存されている（表示が速い）
- **動的サイト**: 表示するたびにデータを集めてページを作る（表示が遅いことがある）

Astroは「静的サイト」を作るためのツールで、**表示が非常に速い**のが特徴です。

## 1-2. なぜAstroを選ぶのか？

### ✅ 高速・軽量
- ページの表示が速い
- サーバーの負担が少ない
- スマートフォンでも快適に動作

### ✅ 学びやすい
- コードがシンプルで読みやすい
- 日本語のドキュメントが充実している
- コミュニティが活発で困った時に助けてくれる

### ✅ 柔軟性が高い
- ReactやVueなどの他の道具も一緒に使える
- 既存の知識を活かせる

### 📊 他のフレームワークとの比較

| フレームワーク | 表示速度 | 学習難易度 | 日本語情報 |
|--------------|----------|-----------|-----------|
| **Astro** | ⚡最速 | 👶やさしい | 👍充実 |
| React       | ⚡速い   | 🤓難しい  | 👍普通   |
| Vue         | ⚡速い   | 👌普通    | 👍充実   |
| Next.js     | ⚡速い   | 🤓難しい  | 👍普通   |

## 1-3. Islands Architecture（アイランドアーキテクチャ）

Astroの最大の特徴である「アイランドアーキテクチャ」を日常的な例えで説明します。

### 🏝️ 海に浮かぶ島をイメージしてください

```
┌─────────────────────────────────────────────┐
│          🌊 静的HTMLの海（速い！）              │
│  ┌──────────┐       ┌──────────┐           │
│  │ 🏝️ 島1   │ 〜〜〜  │ 🏝️ 島2   │           │
│  │ （検索機能）│       │ （コメント）│          │
│  └──────────┘       └──────────┘           │
│                                              │
│  島だけインタラクティブ（JavaScriptあり）      │
└─────────────────────────────────────────────┘
```

- **海（静的HTML）**: ほとんどの部分は静的（高速）
- **島（インタラクティブ部分）**: 必要な部分だけJavaScriptで動く

### なぜ「島」と呼ぶのか？

- 大きな「海」の中に、小さな「島」があるイメージ
- 必要な部分だけ「島」として機能を追加する
- 島は独立していて、それぞれ別の道具（ReactやVueなど）を使える

### メリット

- **高速**: ほとんどは静的HTMLなので表示が速い
- **省エネ**: 必要な部分だけJavaScriptを読み込む
- **柔軟**: 島ごとに違う道具を使える

---

# 2. プロジェクトの作り方

## 2-1. 事前準備

### Node.jsのインストール確認

Astroを使うには、**Node.js**という道具が必要です。Node.jsは「JavaScriptをパソコン上で動かすための道具」です。

**確認方法**:

1. ターミナル（Mac）またはコマンドプロンプト（Windows）を開く
2. 以下のコマンドを入力してEnterキーを押す

```bash
node --version
```

**結果の見方**:

- バージョン番号（例: `v20.10.0`）が表示されればOK ✅
- `command not found` と表示されたらインストールが必要 ⚠️

**インストール方法**（まだ入っていない場合）:

1. [Node.js公式サイト](https://nodejs.org/)にアクセス
2. 「LTS版」（推奨版）をダウンロード
3. インストーラーの指示に従ってインストール

## 2-2. プロジェクト作成コマンド

準備ができたら、さっそくプロジェクトを作りましょう！

### ターミナルで以下を実行

```bash
npm create astro@latest my-first-site
```

**このコマンドの意味**:
- `npm`: Node.jsの道具を使うためのコマンド
- `create`: 新しく作る
- `astro@latest`: Astroの最新版
- `my-first-site`: プロジェクトの名前（好きな名前でOK）

### インストール中の質問に答える

インストール中にいくつか質問されます。おすすめの選択を紹介します。

```
✔ Where should we create your new project?
  › my-first-site

✔ How would you like to start your new project?
  › Include sample files (recommended)

✔ Install dependencies?
  › Yes

✔ Initialize a new git repository?
  › Yes (バージョン管理したい場合)

✔ How would you like to setup TypeScript?
  › Relax (推奨)
```

### プロジェクトを起動する

インストールが完了したら、プロジェクトのフォルダに移動して起動します。

```bash
cd my-first-site
npm run dev
```

**「Local」に表示されたURLをクリックすると、ブラウザでサイトが開きます！**

通常は: `http://localhost:4321`

## 2-3. プロジェクト構成の説明

プロジェクトを作ると、以下のようなフォルダ構成になります。

```
my-first-site/
├── src/              # メインの作業フォルダ
│   ├── pages/        # ページファイル
│   ├── components/   # コンポーネント（後で説明）
│   ├── layouts/      # レイアウト（共通デザイン）
│   └── styles/       # スタイルシート
├── public/           # 画像などの静的ファイル
├── astro.config.mjs  # Astroの設定ファイル
└── package.json      # プロジェクト情報
```

**それぞれのフォルダの役割**:

| フォルダ | 役割 | 例 |
|---------|------|----|
| `src/pages/` | Webサイトのページ | トップページ、Aboutページ |
| `src/components/` | 使い回す部品 | ヘッダー、フッター、カード |
| `src/layouts/` | ページの枠組み | 共通デザイン |
| `public/` | そのまま使うファイル | ロゴ画像、favicon |

---

# 3. ページの作成方法

## 3-1. .astroファイルの基本構造

Astroでは、`.astro`という拡張子のファイルを使ってページを作ります。

### 基本構造

```astro
---
// フロントマター（ここにコードを書く）
const pageTitle = "私の最初のサイト";
---
<!-- HTML（ここにデザインを書く） -->
<html>
  <head>
    <title>{pageTitle}</title>
  </head>
  <body>
    <h1>ようこそ、Astroの世界へ！</h1>
  </body>
</html>
```

### フロントマターとは？

`---`で囲まれた部分を「フロントマター」と呼びます。

**役割**:
- 変数の宣言
- データの取得
- インポート（他のファイルを読み込む）

**例え**: レシピの「材料リスト」のようなもの。料理（HTML）を作る前に準備しておくことを書く場所です。

## 3-2. HTMLの書き方

Astroの中では、普通のHTMLがそのまま使えます。

```html
<h1>見出し1</h1>
<h2>見出し2</h2>
<p>段落テキスト</p>
<a href="/about">リンク</a>
<img src="/image.jpg" alt="画像説明">
```

## 3-3. JavaScriptの書き方

フロントマターの中で、JavaScriptのコードが書けます。

```astro
---
const siteName = "My Site";
const pages = ["Home", "About", "Blog"];

// 繰り返し処理もできる
const pageList = pages.map(page => {
  return { name: page, url: `/${page.toLowerCase()}` };
});
---
```

## 3-4. フロントマターの実践的な使い方

### ページ情報を設定する

```astro
---
// ページのメタデータ
const title = "About Me";
const description = "自己紹介ページです";

// 現在の日付
const today = new Date().toLocaleDateString('ja-JP');
---
<html>
  <head>
    <title>{title}</title>
    <meta name="description" content={description} />
  </head>
  <body>
    <p>今日の日付: {today}</p>
  </body>
</html>
```

### データの一覧表示

```astro
---
const posts = [
  { title: "最初の投稿", date: "2026-01-01" },
  { title: "Astroを学ぶ", date: "2026-01-15" },
  { title: "Web開発の日々", date: "2026-01-30" }
];
---
<ul>
  {posts.map(post => (
    <li>
      <strong>{post.title}</strong> - {post.date}
    </li>
  ))}
</ul>
```

---

# 4. コンポーネントの使い方

## 4-1. コンポーネントとは何か？

**コンポーネント**は「使い回せる部品」のことです。

### 例えで理解する

- ** LEGOブロック**: 同じパーツを組み合わせて様々なものを作る
- **料理の下ごしらえ**: 切った野菜を色々な料理に使う
- **Webサイトの部品**: ヘッダー、フッター、ボタンなど

### メリット

- **使い回せる**: 一度作れば何度でも使える
- **修正が楽**: 一箇所直せば全ての部品が更新される
- **管理しやすい**: 小さな部品ごとに管理できる

## 4-2. コンポーネントの作成

### 基本的なコンポーネント

`src/components/Card.astro`を作成:

```astro
---
// Props（プロパティ）の型定義
const { title, description } = Astro.props;
---
<div class="card">
  <h2>{title}</h2>
  <p>{description}</p>
</div>

<style>
  .card {
    border: 1px solid #ccc;
    padding: 1rem;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  }
</style>
```

## 4-3. コンポーネントの使い回し

### コンポーネントをインポートして使う

```astro
---
// Cardコンポーネントをインポート
import Card from '../components/Card.astro';

// カードのデータ
const items = [
  { title: "カード1", description: "これは1枚目のカードです" },
  { title: "カード2", description: "これは2枚目のカードです" },
  { title: "カード3", description: "これは3枚目のカードです" }
];
---
<div class="card-container">
  {items.map(item => (
    <Card title={item.title} description={item.description} />
  ))}
</div>
```

### 実践例: ブログ記事カード

```astro
---
import PostCard from '../components/PostCard.astro';

const posts = [
  {
    title: "Astro入門",
    excerpt: "Astroの基本的な使い方を紹介します",
    date: "2026-01-30",
    category: "プログラミング"
  },
  {
    title: "Web開発のすすめ",
    excerpt: "個人開発で役立つノウハウ",
    date: "2026-01-28",
    category: "キャリア"
  }
];
---
<div class="posts">
  {posts.map(post => (
    <PostCard
      title={post.title}
      excerpt={post.excerpt}
      date={post.date}
      category={post.category}
    />
  ))}
</div>
```

## 4-4. Propsでデータを渡す方法

**Props（プロプス）**は、コンポーネントにデータを渡すための仕組みです。

### 例え: 注文システム

- コンポーネント = 店員
- Props = 注文内容
- 結果 = 注文通りに作られた料理

### Propsの受け取り方

```astro
---
// Propsを定義
const { name, age, country } = Astro.props;

// デフォルト値を設定することも可能
const { title = "デフォルトタイトル", count = 0 } = Astro.props;
---
<div>
  <p>名前: {name}</p>
  <p>年齢: {age}</p>
  <p>国: {country}</p>
</div>
```

### Propsの渡し方

```astro
---
import UserCard from '../components/UserCard.astro';
---
<!-- Propsを渡してコンポーネントを使う -->
<UserCard
  name="太郎"
  age={25}
  country="日本"
/>
```

---

# 5. ルーティングの仕組み

## 5-1. ファイルベースルーティングとは？

Astroでは、**フォルダ構造がそのままURLになります**。これを「ファイルベースルーティング」と呼びます。

### 具体例

```
src/pages/
├── index.astro        → https://example.com/
├── about.astro        → https://example.com/about
├── blog/
│   ├── index.astro    → https://example.com/blog
│   └── first-post.astro → https://example.com/blog/first-post
└── contact/
    └── index.astro    → https://example.com/contact
```

### 分かりやすいルール

| ファイル名 | URLになる形 |
|-----------|-------------|
| `index.astro` | `/` （そのフォルダのトップ） |
| `about.astro` | `/about` |
| `blog/index.astro` | `/blog` |
| `blog/post.astro` | `/blog/post` |

## 5-2. ページの追加方法

### 新しいページを作る手順

1. `src/pages/`フォルダを開く
2. 新しいファイルを作る（例: `profile.astro`）
3. ファイルを保存する
4. 自動的に `/profile` でアクセスできるようになる

### ページのサンプル

```astro
---
// src/pages/profile.astro
const pageTitle = "プロフィール";
const user = {
  name: "山田太郎",
  bio: "Webエンジニア目初学者です",
  interests: ["プログラミング", "読書", "旅行"]
};
---
<html>
  <head>
    <title>{pageTitle}</title>
  </head>
  <body>
    <h1>{pageTitle}</h1>
    <h2>{user.name}</h2>
    <p>{user.bio}</p>
    <h3>趣味</h3>
    <ul>
      {user.interests.map(interest => <li>{interest}</li>)}
    </ul>
  </body>
</html>
```

## 5-3. ディレクトリ構造とURLの関係

### ネストされたページ

```
src/pages/
└── blog/
    ├── index.astro           → /blog
    ├── 2024/
    │   ├── index.astro       → /blog/2024
    │   └── jan/
    │       └── 15.astro      → /blog/2024/jan/15
    └── about.astro           → /blog/about
```

### 実践例: ブログサイトの構造

```
src/pages/
├── index.astro              # トップページ
├── about.astro              # Aboutページ
├── blog/
│   ├── index.astro          # ブログ一覧
│   ├── [slug].astro         # 個別記事（動的ルーティング）
│   └── category/
│       └── [category].astro # カテゴリ別一覧
└── contact/
    └── index.astro          # お問い合わせ
```

---

# 6. よくあるミスと対処法

## ❌ よくあるミス集

### ミス1: ファイル保存を忘れる

**症状**: 変更したのにブラウザに反映されない

**対処法**:
- ファイルを保存したか確認（`Cmd + S` または `Ctrl + S`）
- Astroの開発サーバーは自動で更新されますが、保存が必要です

### ミス2: コードの閉じ忘れ

**症状**: エラーメッセージが表示される

**例**:
```astro
<!-- ❌ 間違い -->
<div>
  <p>テキスト
<!-- 閉じタグがない -->

<!-- ✅ 正しい -->
<div>
  <p>テキスト</p>
</div>
```

**対処法**:
- エディタの「自動補完」機能を活用する
- コードの色分えを見て、変な箇所がないか確認する

### ミス3: インポート忘れ

**症状**: `XXX is not defined` というエラー

**例**:
```astro
<!-- ❌ 間違い -->
<Card title="タイトル" />
<!-- Cardをインポートしていない -->

<!-- ✅ 正しい -->
---
import Card from '../components/Card.astro';
---
<Card title="タイトル" />
```

**対処法**:
- コンポーネントを使う前に必ずインポートする
- エディタの「自動インポート」機能を使う

### ミス4: ファイルパスの間違い

**症状**: モジュールが見つからないエラー

**対処法**:
- 相対パスの書き方を確認
- ファイルの場所を確認

```
../    # 親フォルダ
./     # 現在のフォルダ
/      # ルート（一番上）から
```

### ミス5: ポートが使われている

**症状**: `Port 4321 is already in use` というエラー

**対処法**:
1. 他のAstroプロジェクトを閉じる
2. または別のポートで起動する:
   ```bash
   npm run dev -- --port 4322
   ```

---

# 7. サンプルコード集

## 7-1. シンプルなカードコンポーネント

```astro
---
// src/components/Card.astro
const { title, content, image } = Astro.props;
---
<article class="card">
  {image && <img src={image} alt={title} />}
  <h3>{title}</h3>
  <p>{content}</p>
</article>

<style>
  .card {
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 1rem;
    background: white;
    transition: transform 0.2s;
  }
  .card:hover {
    transform: translateY(-4px);
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  }
  img {
    width: 100%;
    border-radius: 4px;
  }
</style>
```

## 7-2. ナビゲーションコンポーネント

```astro
---
// src/components/Navigation.astro
const menuItems = [
  { name: "ホーム", url: "/" },
  { name: "About", url: "/about" },
  { name: "ブログ", url: "/blog" },
  { name: "お問い合わせ", url: "/contact" }
];
---
<nav class="navigation">
  <ul>
    {menuItems.map(item => (
      <li>
        <a href={item.url}>{item.name}</a>
      </li>
    ))}
  </ul>
</nav>

<style>
  .navigation {
    background: #333;
  }
  .navigation ul {
    display: flex;
    list-style: none;
    margin: 0;
    padding: 0;
  }
  .navigation li {
    margin: 0 1rem;
  }
  .navigation a {
    color: white;
    text-decoration: none;
    padding: 1rem;
    display: block;
  }
  .navigation a:hover {
    background: #555;
  }
</style>
```

## 7-3. レイアウトテンプレート

```astro
---
// src/layouts/BaseLayout.astro
const { title = "My Site" } = Astro.props;
---
<html lang="ja">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>{title}</title>
  </head>
  <body>
    <header>
      <h1>My Site</h1>
    </header>
    <main>
      <slot />  <!-- ここにページの内容が入る -->
    </main>
    <footer>
      <p>&copy; 2026 My Site</p>
    </footer>
  </body>
</html>

<style>
  body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 0;
  }
  header {
    background: #f0f0f0;
    padding: 1rem 2rem;
  }
  main {
    max-width: 800px;
    margin: 2rem auto;
    padding: 0 1rem;
  }
  footer {
    background: #333;
    color: white;
    text-align: center;
    padding: 1rem;
  }
</style>
```

## 7-4. レイアウトの使い方

```astro
---
// src/pages/index.astro
import BaseLayout from '../layouts/BaseLayout.astro';
---
<BaseLayout title="ホーム">
  <h2>ようこそ！</h2>
  <p>これは私の最初のAstroサイトです。</p>
</BaseLayout>
```

---

# 8. 用語集

| 用語 | 読み方 | 意味 | 例え |
|------|--------|------|------|
| **フレームワーク** | ふれーむわーく | アプリを作るための枠組み、道具箱 | 家具の組み立てキット |
| **静的サイト** | せいてきさいと | あらかじめ作っておいたページ | 印刷されたパンフレット |
| **動的サイト** | どうてきさいと | 表示時にデータを集めて作るページ | 空港の電光掲示板 |
| **Node.js** | のーどじぇいえす | JavaScriptをパソコンで動かす道具 | プログラミング言語の翻訳機 |
| **npm** | えぬぴーえむ | Node.jsのパッケージ（道具）を管理するツール | アプリストア |
| **フロントマター** | ふろんとまたー | ファイルの先頭の設定部分 | レシピの材料リスト |
| **コンポーネント** | こんぽーねんと | 使い回せる部品 | LEGOブロック |
| **Props** | ぷろぷす | コンポーネントに渡すデータ | 注文内容 |
| **ルーティング** | るーてぃんぐ | URLとページの対応付け | 地図の道標 |
| **ファイルベース** | ふぁいるべーす | ファイルの場所でURLが決まる仕組み | ファイル整理のルール |
| **インタラクティブ** | いんたらくてぃぶ | ユーザーの操作に反応すること | ボタンを押すと動く |
| **レイアウト** | れいあうと | ページの共通デザイン | 文書のテンプレート |
| **ホスティング** | ほすてぃんぐ | Webサイトを公開するサービス | 家を建てる土地 |
| **デプロイ** | でぷろい | Webサイトを公開すること | お店をオープンする |
| **ローカル環境** | ろーかるかんきょう | 自分のパソコン上のテスト環境 | キッチンで試作 |
| **本番環境** | ほんばんかんきょう | �際に公開される環境 | 実際の店舗 |

---

# 9. 次のステップ

このガイドを読んだら、次は実際に手を動かしてみましょう！

1. ✅ **Astroプロジェクトを作成する**
   ```bash
   npm create astro@latest my-first-site
   ```

2. ✅ **ページを作成する**
   - `src/pages/index.astro` を編集
   - 新しいページを追加

3. ✅ **コンポーネントを作る**
   - `src/components/` にファイルを作成
   - ページで使ってみる

4. ✅ **スタイルを追加する**
   - `<style>` タグで見た目を変更

5. ✅ **公開する**
   - VercelやNetlifyなどで無料公開

---

# 参考リンク

### 公式ドキュメント
- [Astro公式サイト](https://astro.build/)
- [Astroドキュメント（日本語）](https://docs.astro.build/ja/)
- [Islands Architecture（アイランドアーキテクチャ）](https://docs.astro.build/en/concepts/islands/)

### チュートリアル
- [Astro初心者チュートリアル（CloudCannon）](https://cloudcannon.com/tutorials/astro-beginners-tutorial-series/)
- [Astro入門ガイド（Hygraph）](https://hygraph.com/blog/astro-javascript)
- [Astro 30分チュートリアル](https://jsundefined.com/en/tutorials/astro/path/30min)

### 動画チュートリアル
- [Learn Astro in 2026 - Crash Course for Beginners](https://www.youtube.com/watch?v=brdy8HU03e4)

### 詳細解説
- [Astro Islands Architecture Explained（Strapi）](https://strapi.io/blog/astro-islands-architecture-explained-complete-guide)
- [Understanding Astro Islands Architecture（LogRocket）](https://blog.logrocket.com/understanding-astro-islands-architecture/)

---

**最後に**: プログラミングは「失敗から学ぶ」ものです。エラーが出ても慌てず、一つ一つ解決していきましょう。応援しています！
