# today i learned Next.js

according to [this Tutrial](https://nextjs.org/learn/basics/create-nextjs-app?utm_source=next-site&utm_medium=homepage-cta&utm_campaign=next-website) published by Vercel.

<!-- TOC -->

- [today i learned Next.js](#today-i-learned-nextjs)
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
    - [Two Forms of Pre-rendering](#two-forms-of-pre-rendering)
    - [Static Generation with and without Data](#static-generation-with-and-without-data)
    - [Implement getStaticProps](#implement-getstaticprops)
    - [getStaticProps Details](#getstaticprops-details)
      - [Fetching Data at Request Time](#fetching-data-at-request-time)
      - [SWR](#swr)
  - [Dynamic Routes](#dynamic-routes)
    - [Page Path Depends on External Data](#page-path-depends-on-external-data)
    - [Render Markdown](#render-markdown)

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
React Router が必要ないのはありがたい。  
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

<img src="https://github.com/fujiokayu/next-til/blob/master/public/images/no-pre-rendering.png" width="300">
<img src="https://github.com/fujiokayu/next-til/blob/master/public/images/pre-rendering.png" width="300">

### Two Forms of Pre-rendering

Next.js のプリレンダリングは２種類あり、Static Generation と SSR がある。  
※ development mode だと全て Static Generation されるという罠がある。  
重要なのはページごとにどのプリレンダリングフォームを使用するかを選択できることであり、これによってハイブリッドな Next.js アプリを作成することができる。  
基本的には Static Generation を使っていればパフォーマンスが良い。一方で、インタラクティブな処理は SSR で実装するのが良いだろう。

### Static Generation with and without Data

Static Generation はアプリがビルドされた時に自動的に生成される。  
ファイルシステムにアクセスしたり、外部 API からデータを取得したり、ビルド時にデータベースに問い合わせたりする場合は、[getStaticProps](https://nextjs.org/docs/basic-features/data-fetching#getstaticprops-static-generation) という非同期関数をエクスポートすることで対応する。

```Javascript
export default function Home(props) { ... }

export async function getStaticProps() {
  // Get external data from the file system, API, DB, etc.
  const data = ...

  // The value of the `props` key will be
  //  passed to the `Home` component
  return {
    props: ...
  }
}
```

### Implement getStaticProps

See [this commit](https://github.com/fujiokayu/next-til/commit/bf2218a678ce39821c6f31f173eff0966033969e)

### getStaticProps Details

前項ではファイルシステムからデータを取得する getSortedPostsData を実装したが、getStaticProps はサーバー側でのみ実行されるため、もちろん外部 API なども利用できる。  
ブラウザ用の JS バンドルにも含まれず、ブラウザに送信されることなく、データベースへの直接問い合わせのようなコードを書くことが可能。

```Javascript
import fetch from 'node-fetch'

export async function getSortedPostsData() {
  // Instead of the file system,
  // fetch post data from an external API endpoint
  const res = await fetch('..')
  return res.json()
}
```

```Javascript
import someDatabaseSDK from 'someDatabaseSDK'

const databaseClient = someDatabaseSDK.createClient(...)

export async function getSortedPostsData() {
  // Instead of the file system,
  // fetch post data from a database
  return databaseClient.query('SELECT posts...')
}
```

- 注意事項
  - 開発環境 (npm run dev や yarn dev) では、getStaticProps はリクエストごとに実行され、本番環境では、getStaticProps はビルド時に実行される
    - つまり本番環境ではクエリパラメータや HTTP ヘッダなど、リクエスト時にしか利用できないデータを使用することはできない  
  - React はページがレンダリングされる前に必要なデータをすべて持っている必要があるため、getStaticProps はページからしかエクスポートできない
    - そのような場合は、SSR を試すか、プリレンダリングをスキップすることができる

#### Fetching Data at Request Time

SSR を使用するには、getStaticProps の代わりに getServerSideProps をエクスポートする必要があります。

```Javascript
export async function getServerSideProps(context) {
  return {
    props: {
      // props for your component
    }
  }
}
```

getServerSideProps はリクエスト時に呼び出されるので、そのパラメータ(context)はリクエスト固有のパラメータを含む。  
Time to first byte (TTFB) は getStaticProps よりも遅くなるため、リクエスト時にデータを取得しなければならないページをプリレンダリングする必要がある場合にのみ getServerSideProps を使用すること。  

事前に生成しておくことができないケースでは、Client-side Rendering で動的にレンダリングすると良い。

#### SWR

Next.js が開発したデータフェッチ用の React フック。  
キャッシング、再検証、フォーカストラッキング、インターバルでのリフェッチなどを処理する。

```Javascript
import useSWR from 'swr'

function Profile() {
  const { data, error } = useSWR('/api/user', fetch)

  if (error) return <div>failed to load</div>
  if (!data) return <div>loading...</div>
  return <div>hello {data.name}!</div>
}
```

[more info](https://swr.vercel.app/)

## Dynamic Routes

### Page Path Depends on External Data

See [this commit](https://github.com/fujiokayu/next-til/commit/660044032e112461f5e04d1f7ddc9fad24b3edcf)
<img src="https://github.com/fujiokayu/next-til/blob/master/public/images/dynamic-routes.png" width="300">

### Render Markdown

markdown コンテンツのレンダリングには [remark](https://github.com/remarkjs/remark) ライブラリを使う。  
`npm install remark remark-html`

```Javascript
import remark from 'remark'
import html from 'remark-html'
```

```Javascript
export async function getPostData(id) {
  const fullPath = path.join(postsDirectory, `${id}.md`)
  const fileContents = fs.readFileSync(fullPath, 'utf8')

  // Use gray-matter to parse the post metadata section
  const matterResult = matter(fileContents)

  // Use remark to convert markdown into HTML string
  const processedContent = await remark()
    .use(html)
    .process(matterResult.content)
  const contentHtml = processedContent.toString()

  // Combine the data with the id and contentHtml
  return {
    id,
    contentHtml,
    ...matterResult.data
  }
}
```

```Javascript
export default function Post({ postData }) {
  return (
    <Layout>
      {postData.title}
      <br />
      {postData.id}
      <br />
      {postData.date}
      <br />
      <div dangerouslySetInnerHTML={{ __html: postData.contentHtml }} />
    </Layout>
  )
}
```

dangerouslySetInnerHTML 使うのは嫌だ・・・

---

This is a [Next.js](https://nextjs.org/) project bootstrapped with [`create-next-app`](https://github.com/vercel/next.js/tree/canary/packages/create-next-app).
