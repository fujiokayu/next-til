# today i learned NEXT.js

according to [this Tutrial](https://nextjs.org/learn/basics/create-nextjs-app?utm_source=next-site&utm_medium=homepage-cta&utm_campaign=next-website) published by Vercel.

<!-- TOC -->

- [today i learned NEXT.js](#today-i-learned-nextjs)
  - [Create a Next.js App](#create-a-nextjs-app)
  - [Navigate Between Pages](#navigate-between-pages)
  - [Assets, Metadata, and CSS](#assets-metadata-and-css)
    - [Assets](#assets)
    - [Metadata](#metadata)
    - [CSS Styling](#css-styling)
    - [Layout Component](#layout-component)
    - [Global Styles](#global-styles)
  - [Pre-rendering and Data Fetching](#pre-rendering-and-data-fetching)
    - [Pre-rendering](#pre-rendering)

<!-- /TOC -->

## Create a Next.js App

npx などで `create-next-app` を実行してテンプレートを作成でき、この時に様々な Example を選ぶことができる。  

- 参照: [Create Next Appのexampleまとめ](https://qiita.com/takeyuichi/items/a3a2eb2607368eda62fd)  

with-firebase を使えばバックエンドとの結合も爆速で実装できそう。

## Navigate Between Pages

Link コンポーネントを使ってページング処理を実装できる。

```Javascript
import Link from 'next/link'
//〜〜〜〜
Read <Link href="/posts/first-post"><a>this page!</a></Link>
```

```Javascript
import Link from 'next/link'

export default function FirstPost() {
  return (
    <>
      <h1>First Post</h1>
      <h2>
        <Link href="/">
          <a>Back to home</a>
        </Link>
      </h2>
    </>
  )
}
```

関数コンポーネントによるシンプルな実装だが、`<Link href="uri"><a> alias </a></Link>` という書式でページ遷移のように扱える。  
実際には JavaScript の世界で完結しており(client-side navigation)、そのためパフォーマンスに優れているとのこと。

## Assets, Metadata, and CSS

### Assets

サンプルの index.js では

```Javascript
<img src="/vercel.svg" alt="Vercel Logo" className="logo" />
```

と実装しているが、これはトップレベルにある `public/vercel.svg` を参照しており、public をルートとして Asset を参照できる。  
なお、トップの public 配下には robots.txtを置いても良い。

### Metadata

HTML の `<head>` タグの代わりに React Component の `<Head>` を使用する

```Javascript
import Head from 'next/head'

<Head>
  <title>Create Next App</title>
  <link rel="icon" href="/favicon.ico" />
</Head>
```

### CSS Styling

[styled-jsx](https://github.com/vercel/styled-jsx) というライブラリをビルトインでサポートしている。  
もちろん、styled-components や emotion を使用することも可能。

### Layout Component

トップレベルディクトりに `components` フォルダを作成し、そこに layout.js を作成する。  

```Javascript
function Layout({ children }) {
  return <div>{children}</div>
}

export default Layout
```

[CSS Modules](https://nextjs.org/docs/basic-features/built-in-css-support#adding-component-level-css) を使って Layout コンポーネントに CSS を適用する。

- layout.module.css

```Javascript
.container {
  max-width: 36rem;
  padding: 0 1rem;
  margin: 3rem auto 6rem;
}
```

layout.js で layout.module.css を使用する。

```Javascript
import styles from './layout.module.css'

export default function Layout({ children }) {
  return <div className={styles.container}>{children}</div>
}
```

pages/posts/first-post.js のコンポーネントを Layout タグでラップする。  

```Javascript
import Head from 'next/head'
import Link from 'next/link'
import Layout from '../../components/layout'

export default function FirstPost() {
  return (
    <Layout>
      <Head>
        <title>First Post</title>
      </Head>
      <h1>First Post</h1>
      <h2>
        <Link href="/">
          <a>Back to home</a>
        </Link>
      </h2>
    </Layout>
  )
}
```

Chrome dev tools などで HTML を見ると、CSS Modules によって自動的に作成されたユニークな classname が付与されていることがわかる。

### Global Styles

[global CSS](https://nextjs.org/docs/basic-features/built-in-css-support#adding-a-global-stylesheet) をロードするため、`pages` フォルダに `_app.js` を作成する。

```Javascript
export default function App({ Component, pageProps }) {
  return <Component {...pageProps} />
}
```

- global CSS は `pages/_app.js` のみがロードできる
- global CSS は どこにどんな名前で置いても良い
- ここではトップレベルに `styles` フォルダを作り、そこに `global.css` というファイル名にする
- _app.js にインポートする: `import '../styles/global.css'`

## Pre-rendering and Data Fetching

### Pre-rendering

Next.js はデフォルトですべてのページをプリレンダリングする（プリレンダリングは React には備わっていない）。  
これは、Next.js が各ページの HTML を事前に生成することを意味し、クライアント側のJavaScriptですべての処理を行うのではない。  
プリレンダリングを行うことで、パフォーマンスとSEOを向上させることができる。

![no-pre-rendering.png]](./poblic/images/no-pre-rendering.png)

![pre-rendering.png]](./poblic/images/pre-rendering.png)


---

This is a [Next.js](https://nextjs.org/) project bootstrapped with [`create-next-app`](https://github.com/vercel/next.js/tree/canary/packages/create-next-app).
