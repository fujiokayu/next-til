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
    - [Polishing the Post Page](#polishing-the-post-page)
      - [Adding title to the Post Page](#adding-title-to-the-post-page)
      - [Formatting the Date](#formatting-the-date)
      - [Adding CSS](#adding-css)
    - [Polishing the Index Page](#polishing-the-index-page)
    - [Dynamic Routes Details](#dynamic-routes-details)
      - [Fetch External API or Query Database](#fetch-external-api-or-query-database)
      - [Development v.s. Production](#development-vs-production)
      - [Fallback](#fallback)
      - [Catch-all Routes](#catch-all-routes)
      - [Router](#router)
      - [404 Pages](#404-pages)
      - [More Examples](#more-examples)
  - [API Routes](#api-routes)
    - [Creating API Routes](#creating-api-routes)
    - [Creating a simple API endpoint](#creating-a-simple-api-endpoint)
    - [API Routes Details](#api-routes-details)
      - [Do Not Fetch an API Route from getStaticProps or getStaticPaths](#do-not-fetch-an-api-route-from-getstaticprops-or-getstaticpaths)
      - [A Good Use Case: Handling Form Input](#a-good-use-case-handling-form-input)
      - [Preview Mode](#preview-mode)
      - [Dynamic API Routes](#dynamic-api-routes)
  - [Deploying Your Next.js App](#deploying-your-nextjs-app)
    - [Push to GitHub](#push-to-github)
    - [Deploy to Vercel](#deploy-to-vercel)
    - [Next.js and Vercel](#nextjs-and-vercel)
      - [Preview Deployment for Every Push](#preview-deployment-for-every-push)
    - [Other Hosting Options](#other-hosting-options)
    - [Finally](#finally)
      - [Join the Conversation](#join-the-conversation)

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

トップレベルディクトリに `components` フォルダを作成し、そこに layout.js を作成する。  

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

dangerouslySetInnerHTML を使うのは嫌だ・・・

### Polishing the Post Page

#### Adding title to the Post Page

```Javascript
// Add this line to imports
import Head from 'next/head'

export default function Post({ postData }) {
  return (
    <Layout>
      <Head>
        <title>{postData.title}</title>
      </Head>
      ...
    </Layout>
  )
}
```

#### Formatting the Date

`npm install date-fns`

create the Date component at components/date.js

```Javascript
import { parseISO, format } from 'date-fns'

export default function Date({ dateString }) {
  const date = parseISO(dateString)
  return <time dateTime={dateString}>{format(date, 'LLLL d, yyyy')}</time>
}
```

```Javascript
// Add this line to imports
import Date from '../../components/date'

export default function Post({ postData }) {
  return (
    <Layout>
      ...
      {/* Replace {postData.date} with this */}
      <Date dateString={postData.date} />
      ...
    </Layout>
  )
}
```

今まで Moment を使っていたので date-fns を知らなかったのだけれど、これは良さそう。(Next 関係ない)  
この開発者は Moment に対して問題を感じてこのライブラリを開発したそうだ。

- 参考: [面倒なJavaScriptの日付の処理は「date-fns」でラクに片付けよう](https://www.webprofessional.jp/date-fns-javascript-date-library/)

#### Adding CSS

```Javascript
// Add this line
import utilStyles from '../../styles/utils.module.css'

export default function Post({ postData }) {
  return (
    <Layout>
      <Head>
        <title>{postData.title}</title>
      </Head>
      <article>
        <h1 className={utilStyles.headingXl}>{postData.title}</h1>
        <div className={utilStyles.lightText}>
          <Date dateString={postData.date} />
        </div>
        <div dangerouslySetInnerHTML={{ __html: postData.contentHtml }} />
      </article>
    </Layout>
  )
}
```

### Polishing the Index Page

Link コンポーネントを使って各ページへのリンクを貼る場合は以下のように実装する。

```Javascript
<Link href="/posts/[id]" as="/posts/ssg-ssr">
  <a>...</a>
</Link>
```

href タグに[id]を渡し、as props に ssg-ssr へのパスを指定する必要がある。
pages/index.js に以下を追加し、

```Javascript
import Link from 'next/link'
import Date from '../components/date'
```

Home コンポーネントの `<li>` タグを以下に書き換える。

```Javascript
<li className={utilStyles.listItem} key={id}>
  <Link href="/posts/[id]" as={`/posts/${id}`}>
    <a>{title}</a>
  </Link>
  <br />
  <small className={utilStyles.lightText}>
    <Date dateString={date} />
  </small>
</li>
```

### Dynamic Routes Details

#### Fetch External API or Query Database

getAllPostIds で外部 API からデータを取得するサンプル

```Javascript
export async function getAllPostIds() {
  // Instead of the file system,
  // fetch post data from an external API endpoint
  const res = await fetch('..')
  const posts = await res.json()
  return posts.map(post => {
    return {
      params: {
        id: post.id
      }
    }
  })
}
```

#### Development v.s. Production

- In development (npm run dev or yarn dev), getStaticPaths runs on every request.
- In production, getStaticPaths runs at build time.

#### Fallback

getStaticPaths から fallback: false がリターンされた場合、指定されたパスが 404 not found。  

- fallback is true の場合、パスがリターンされてビルド時に HTML がレンダリングされる。
- false でないにも関わらずパスが生成されなかった場合は 404 ではなく、Next.js は fallback version を提供する？
  - ここら辺は [fallback documentation](https://nextjs.org/docs/basic-features/data-fetching#the-fallback-key-required) をちゃんと読もう

#### Catch-all Routes

pages/posts/[...id].js matches /posts/a, but also /posts/a/b, /posts/a/b/c and so on.  
[https://nextjs.org/docs/routing/dynamic-routes#catch-all-routes](https://nextjs.org/docs/routing/dynamic-routes#catch-all-routes)

#### Router

[useRouter](https://nextjs.org/docs/api-reference/next/router#userouter) という hook がある。  
Link コンポーネントがなくても必要なのか・・・？

#### 404 Pages

pages/404.js カスタム 404 ページを作れる。

#### More Examples

- [Blog Starter using markdown files](https://github.com/vercel/next.js/tree/canary/examples/blog-starter)
- [WordPress Example](https://github.com/vercel/next.js/tree/canary/examples/cms-wordpress)
- [DatoCMS Example](https://github.com/vercel/next.js/tree/canary/examples/cms-datocms)
- [TakeShape Example](https://github.com/vercel/next.js/tree/canary/examples/cms-takeshape)
- [Sanity Example](https://github.com/vercel/next.js/tree/canary/examples/cms-sanity)

## API Routes

### Creating API Routes

[API Routes](https://nextjs.org/docs/api-routes/introduction) を使用すると、Next.js アプリ内に API エンドポイントを作成することができる。  
これは pages/api ディレクトリ内に関数を作成することで作成し、Serverless Functions としてデプロイできる。  

### Creating a simple API endpoint

pages/api/hello.js

```Javascript
export default (req, res) => {
  res.status(200).json({ text: 'Hello' })
}
```

req は [http.IncomingMessage](https://nodejs.org/api/http.html#http_class_http_incomingmessage) の、res は [http.ServerResponse](https://nodejs.org/api/http.html#http_class_http_serverresponse) のインスタンス。

### API Routes Details

#### Do Not Fetch an API Route from getStaticProps or getStaticPaths

getStaticProps と getStaticPaths はサーバーサイドでのみ実行されるため、API Route を getStaticProps や getStaticPaths から取得してはいけない。  
代わりに、サーバーサイドのコードを直接 getStaticProps や getStaticPaths に記述するか、ヘルパー関数を呼び出すこと

#### A Good Use Case: Handling Form Input

API Routes の良いユースケースはフォーム入力の処理。  
例えばページ上にフォームを作成し、そのフォームからAPIルートにPOSTリクエストを送信させることができ、それを直接データベースに保存するコードを書くことができる。  
API Route コードはクライアントバンドルの一部ではないので、サーバーサイドのコードを安全に書くことができる。

```Javascript
export default (req, res) => {
  const email = req.body.email
  // Then save email to your database, etc...
}
```

#### Preview Mode

[Static Generation](https://nextjs.org/docs/basic-features/pages#static-generation-recommended) はページがヘッドレス CMS からデータを取得する場合に便利だが、ヘッドレス CMS で下書きを書いていて、その下書きをすぐにページ上でプレビューしたい場合には理想的ではない。  
Next.js ではこれらのページをビルド時ではなくリクエスト時にレンダリングし、公開されたコンテンツではなくドラフトコンテンツを取得するようにしたいと思う場合に限って、Next.js はスタティック生成をバイパスする。

#### Dynamic API Routes

API Routes は通常のページと同様に動的に設定することができる。詳しくは [Dynamic API Routes のドキュメント](https://nextjs.org/docs/api-routes/dynamic-api-routes)

## Deploying Your Next.js App

### Push to GitHub

リポジトリはパブリックでもプライベートでも OK。

### Deploy to Vercel

Next.js を本番環境にデプロイする最も簡単な方法は、Next.js の開発者が開発した Vercel プラットフォームを利用すること。  
Vercel はスタティック＆JAMstack デプロイをサポートするグローバル CDN とサーバーレス機能を備えたオールインワンのプラットフォーム。  
クレジットカード不要で無料で利用できる。  

1. まず、https://vercel.com/signup にアクセスして、Vercel のアカウントを作成。  
1. Continue with GitHub を選択し、サインアップの手続きを行う。（Full Name を入力するくらい）
1. リポジトリの URL を指定。これだったら https://github.com/fujiokayu/next-til
1. GitHub 用に Vercel をインストールする。この時に前リポジトリにアクセス権を与えるか、個別のリポジトリにするか設定できる
1. デプロイの設定はほぼ自動で推測してくれたのでそのまま Deploy
1. デプロイ後に公開リンクが表示される。 https://next-til.vercel.app/

### Next.js and Vercel

Next.js アプリを Vercel にデプロイするとデフォルトで以下のようになる。  

- 静的生成とアセット（JS、CSS、画像、フォントなど）を使用するページは、自動的に Vercel Edge Network から提供される
- サーバーサイドレンダリングと API ルートを使用するページは、自動的に分離されたサーバーレス関数になる
  - これにより、ページのレンダリングや API リクエストを無限にスケールさせることができる
  
Vercelには他にも以下のような機能がある。

- Custom Domains
  - Vercel にデプロイすると Next.js アプリにカスタムドメインを割り当てることができる。see [this documentation](https://vercel.com/docs/custom-domains)
- Environment Variables
  - Vercel 上で環境変数を設定することができる。see [this documentation](https://vercel.com/docs/build-step#environment-variables)
- Automatic HTTPS
  - HTTPS はデフォルトで有効になっており（カスタムドメインを含む）、特別な設定は必要なく、SSL 証明書は自動更新される。
  
[more information](https://vercel.com/docs)

#### Preview Deployment for Every Push

Vercel にデプロイした後、新しいプルリクエストを送ると Vercel bot がその変更を適用したプレビュー画面の URL をコメントしてくれる

### Other Hosting Options

Next.js は Node.js をサポートしているホスティングプロバイダであればどこにでもデプロイすることができる。  
ここまでの手順を踏めば、package.jsonにはビルドスクリプトとスタートスクリプトが含まれているはず  

```Json
{
  "scripts": {
    "dev": "next",
    "build": "next build",
    "start": "next start"
  }
}
```

`npm run build` でビルドすると、.next フォルダ内に本番アプリケーションがビルドされる。  
`npm run start` でハイブリッドページをサポートする Node.js サーバが起動され、静的に生成されたページとサーバサイドでレンダリングされたページの両方を提供し、APIルートを提供する  。
  
package.json のスタートスクリプトを以下のように編集するとポートも指定できる。
`"start": "next start -p $PORT"`

### Finally

- TypeScript を使いたい場合は [see this document](https://nextjs.org/learn/excel/typescript)
- 次に学ぶことのおすすめは以下
  - [Data Fetching](https://nextjs.org/docs/basic-features/data-fetching)
  - [Environment Variables](https://nextjs.org/docs/basic-features/environment-variables)
  - [AMP Support](https://nextjs.org/docs/advanced-features/amp-support/introduction)

#### Join the Conversation

Next.js に質問があれば [GitHub Discussions](https://github.com/vercel/next.js/discussions) へ

---

This is a [Next.js](https://nextjs.org/) project bootstrapped with [`create-next-app`](https://github.com/vercel/next.js/tree/canary/packages/create-next-app).
