# today i learned NEXT.js

according to [this Tutrial](https://nextjs.org/learn/basics/create-nextjs-app?utm_source=next-site&utm_medium=homepage-cta&utm_campaign=next-website) published by Vercel.

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

### Assets, Metadata, and CSS

#### Assets

サンプルの index.js では

```Javascript
<img src="/vercel.svg" alt="Vercel Logo" className="logo" />
```

と実装しているが、これはトップレベルにある `public/vercel.svg` を参照している。public をルートとして Asset を参照できる。
なお、トップの public 配下には robots.txtを置いても良い。

#### Metadata

HTML の `<head>` タグの代わりに React Component の `<Head>` を使用する

```Javascript
<Head>
  <title>Create Next App</title>
  <link rel="icon" href="/favicon.ico" />
</Head>
```

---

This is a [Next.js](https://nextjs.org/) project bootstrapped with [`create-next-app`](https://github.com/vercel/next.js/tree/canary/packages/create-next-app).

## Getting Started

First, run the development server:

```bash
npm run dev
# or
yarn dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

You can start editing the page by modifying `pages/index.js`. The page auto-updates as you edit the file.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js/) - your feedback and contributions are welcome!

## Deploy on Vercel

The easiest way to deploy your Next.js app is to use the [Vercel Platform](https://vercel.com/import?utm_medium=default-template&filter=next.js&utm_source=create-next-app&utm_campaign=create-next-app-readme) from the creators of Next.js.

Check out our [Next.js deployment documentation](https://nextjs.org/docs/deployment) for more details.
