---
title: Next.js アプリを IndexNow で高速インデックス化する方法
date: 2024-08-07T15:33:29.357Z
author: Vivek Sahu
authorURL: https://www.freecodecamp.org/news/author/wewake/
originalURL: https://www.freecodecamp.org/news/how-to-index-nextjs-pages-with-indexnow/
translator: ""
reviewer: ""
---

Next.js は、非常に高速なアプリケーションを構築するための強力なフレームワークです。しかし、これらのアプリケーションを検索エンジンに早くインデックスさせることは、可視性とトラフィックのために非常に重要です。しかし、残念ながらこれは即座には実現しません。

<!-- more -->

サイトマップをアップロードした後でも、検索エンジンがあなたのページをクロールするまでに数週間から数ヶ月かかることがあります。特に新しいページを追加したり、既存のページを更新したりした場合、検索エンジンにその変更が認識されるまでにはかなりの時間がかかることがあります。

さて、もっと良い方法はないのでしょうか？

この記事では、IndexNow を使用して、Bing や Yahoo などの主要な検索エンジンであなたの Next.js アプリの SEO を高速化する方法を学びます。

## 目次

-   [IndexNow とは？][1]
    -   [どのように動作するのか？][2]
-   [手順][3]
    -   [前提条件][4]
    -   [高レベルの手順][5]
    -   [ホストの所有権を証明する方法][6]
    -   [サイトマップから全ての URL を取得するスクリプトを作成する方法][7]
    -   [IndexNow API を呼び出す方法][8]
-   [次のステップと改善点][9]
-   [結論][10]
-   [リンクと参考文献][11]

## IndexNow とは？

[IndexNow][12] はインデックス時間を劇的に短縮するプロトコルです。公式ウェブサイトでは次のように定義されています：

> IndexNow は、ウェブサイトのオーナーが最新のコンテンツ変更を検索エンジンに即座に知らせる簡単な方法です。最もシンプルな形では、IndexNow は単なる ping であり、URL とそのコンテンツが追加された、更新された、または削除されたことを検索エンジンに知らせ、これにより検索エンジンは検索結果にこの変更を迅速に反映することができます。([ソース: IndexNow ホーム][13])

このプロトコルは、Bing、Naver、Seznam.cz、Yandex、Yep などによって採用されています。この記事を書いている時点では、Google はこのプロトコルをサポートしていません。

Wix などの多くの CMS にネイティブに統合されていますが、Drupal や WordPress のような他のものにはサードパーティのプラグインがあります。しかし、NextJS にはネイティブサポートがありません。

### どのように動作するのか？

何かを更新するたびに、API を呼び出してこの変更を知らせるだけです。

この情報が受け取られると、検索エンジンは他の自然にクロールされる URL よりも優先してこれらの URL をクロールすることができます。

このガイドでは、既存の Next.js アプリに IndexNow を統合し、URL に変更があった場合に検索エンジンに提出してインデックス化させる方法を紹介します。

### 前提条件

-   Next.js アプリ
-   Next.js アプリ用の **sitemap.xml**

### 高レベルの手順

1.  最初に、URL を提出するためのホストの「所有権を証明」する必要があります。
2.  サイトマップから全ての URL を取得するためのシンプルな Node.js スクリプトを作成します。
3.  IndexNow API を呼び出します。

### ホストの所有権を証明する方法

Bing の [`IndexNow`][14] [ページ][15] へ移動します。Next.js への直接の統合はないので、[手動での統合][16] セクションまでスクロールします。

「Generate」をクリックして新しい API キーを生成します。

Next.js アプリのルートにある **public** ディレクトリに移動します。すべての静的コンテンツはこのディレクトリを通じてレンダリングされます。新しいファイルを作成し、この API キーを保存します：

```bash
# API キーが "f34f184d10c049ef99aa7637cdc4ef04" であると仮定します。生成された API キーに応じて変更してください
echo "f34f184d10c049ef99aa7637cdc4ef04" > f34f184d10c049ef99aa7637cdc4ef04.txt
```

Next.js アプリをビルドして実行します：

```bash
npm run build && npm run start
```

その後、パス `/f34f184d10c049ef99aa7637cdc4ef04.txt` にファイルが存在することを確認します。

つまり、[https://localhost:3000/f34f184d10c049ef99aa7637cdc4ef04.txt][17] を開くと、ブラウザに「f34f184d10c049ef99aa7637cdc4ef04」のテキストが表示されるはずです。

API キーに応じて上記のキーの値を変更します。この変更をコミットし、プッシュして本番環境にデプロイします。

デプロイが成功したら、`<Your URL>/<API Key>.txt` が `<API Key>` テキストをレンダリングすることを確認します。つまり、`<Your URL>/f34f184d10c049ef99aa7637cdc4ef04.txt` が `f34f184d10c049ef99aa7637cdc4ef04` をレンダリングすることを確認します。

### サイトマップから全ての URL を取得するスクリプトを作成する方法

まず、Node スクリプトファイルを作成します：

```bash
touch lib/indexnow.js
```

次に、以下のコードを追加します：

```js
const xml2js = require('xml2js');

// 設定
const sitemapUrl = '<Your URL>/sitemap.xml'; // TODO: 更新
const host = '<Your URL>'; // TODO: 更新
const key = '<API Key>'; // TODO: 更新
const keyLocation = 'https://<Your URL>/<API Key>.txt'; // TODO: 更新

const modifiedSinceDate = new Date(process.argv[2] || '1970-01-01');

if (isNaN(modifiedSinceDate.getTime())) {
  console.error('無効な日付が提供されました。YYYY-MM-DD フォーマットを使用してください');
  process.exit(1);
}

function fetchSitemap(url) {
  return new Promise((resolve, reject) => {
    https.get(url, (res) => {
      let data = '';
      res.on('data', (chunk) => {
        data += chunk;
      });
      res.on('end', () => {
        resolve(data);
      });
    }).on('error', (err) => {
      reject(err);
    });
  });
}
```

```js
function filterUrlsByDate(sitemap, date) {
  const urls = sitemap.urlset.url;
  return urls
    .filter(url => new Date(url.lastmod[0]) > date)
    .map(url => url.loc[0]);
}


async function main() {
  try {
    const xmlData = await fetchSitemap(sitemapUrl);
    const sitemap = await parseSitemap(xmlData);
    const filteredUrls = filterUrlsByDate(sitemap, modifiedSinceDate);
    console.log(filteredUrls);
  } catch (error) {
    console.error('An error occurred:', error);
  }
}

main();
```

このスクリプトは、指定した日付以降に更新されたサイトマップの URL を取得するものです。日付はスクリプトに引数として渡すことができます。

次に、XML レスポンスを解析するために `xml2js` ライブラリをインストールします。

```bash
npm install xml2js --save-dev
```

続いて、スクリプトを実行し、URL が正常に取得できるか確認しましょう。

```bash
node lib/indexnow.js 2024-01-01
```

これで、2024 年 1 月 1 日以降に更新された URL のリストが出力されるはずです。任意の日付を渡すことが可能です。

### IndexNow API を呼び出す方法

以下に IndexNow API のスキーマを示します。

**リクエスト:**

```http
POST /IndexNow HTTP/1.1
Content-Type: application/json; charset=utf-8
Host: api.indexnow.org
{
  "host": "www.example.org",
  "key": "7e3f6e8bc47b4f2380ba54aab6088521",
  "keyLocation": "https://www.example.org/7e3f6e8bc47b4f2380ba54aab6088521.txt",
  "urlList": [
      "https://www.example.org/url1",
      "https://www.example.org/folder/url2",
      "https://www.example.org/url3"
      ]
}
```

**レスポンス:**

| HTTP コード | レスポンス | 理由 |
| --- | --- | --- |
| 200 | Ok | URL が正常に送信されました |
| 400 | Bad request | 無効な形式 |
| 403 | Forbidden | キーが無効（例：キーが見つからない、ファイルがあるがキーがファイルにない） |
| 422 | Unprocessable Entity | URL がホストに属さない、またはプロトコル内のスキーマにキーが一致しない |
| 429 | Too Many Requests | リクエストが多すぎる（スパムの可能性） |

URL 取得部分が正確に動作していることを確認したら、IndexNow API を呼び出すメイン関数を追加しましょう。

お気に入りの IDE を使用して `lib/index.js` ファイルを開き、次の関数を追加します。

```js
function postToIndexNow(urlList) {
  const data = JSON.stringify({
    host,
    key,
    keyLocation,
    urlList
  });

  const options = {
    hostname: 'api.indexnow.org',
    port: 443,
    path: '/IndexNow',
    method: 'POST',
    headers: {
      'Content-Type': 'application/json; charset=utf-8',
      'Content-Length': data.length
    }
  };

  return new Promise((resolve, reject) => {
    const req = https.request(options, (res) => {
      let responseData = '';
      res.on('data', (chunk) => {
        responseData += chunk;
      });
      res.on('end', () => {
        resolve({
          statusCode: res.statusCode,
          statusMessage: res.statusMessage,
          data: responseData
        });
      });
    });

    req.on('error', (error) => {
      reject(error);
    });

    req.write(data);
    req.end();
  });
}
```

この関数は、URL のリストと `<API Key>` を渡して IndexNow API を呼び出します。

メイン関数からこの関数を呼び出すよう修正します。メイン関数は次のように変更します。

```js
async function main() {
  try {
    const xmlData = await fetchSitemap(sitemapUrl);
    const sitemap = await parseSitemap(xmlData);
    const filteredUrls = filterUrlsByDate(sitemap, modifiedSinceDate);
    console.log(filteredUrls);
    
    if (filteredUrls.length > 0) {
      const response = await postToIndexNow(filteredUrls);
      console.log('IndexNow API Response:');
      console.log('Status:', response.statusCode, response.statusMessage);
      console.log('Data:', response.data);
    } else {
      console.log('指定された日付以降に更新された URL はありません。');
    }
  } catch (error) {
    console.error('エラーが発生しました:', error);
  }
}
```

これで、指定された日付以降に更新されたすべての URL に対して IndexNow API が呼び出されます。

スクリプトを実行すると、成功時には以下のような出力が表示され、エラーが発生した場合にはエラーメッセージが表示されます。

```bash
% node lib/indexnow.js 2024-07-24

[
  'https://<Your URL>',
  'https://<Your URL>/page1',
  'https://<Your URL>/page1'
]
IndexNow API Response:
Status: 200 OK
Data:
```

これで、API が IndexNow と連携して高速なインデックス作成が可能になります。

## 次のステップと改善点

今回のスクリプトは、ローカル環境で実行することで IndexNow を介してページをインデックス化する方法を紹介しました。さらなる改善点として以下のような方法があります。

- CI/CD パイプラインに IndexNow を統合し、自動更新を実現。
- 大規模なサイトマップを効率的に処理するために、バッチ処理やキューを使用。
- IndexNow 送信を監視し、デバッグや分析のためのログを生成。
- IndexNow API の追加機能を探求（例：URL 削除）。
- CLI バージョンの提供。
- TypeScript サポートの追加。

これらは本記事の範囲外ですが、いくつかの npm モジュールは、これらの高度な機能を実装したものであり、それらをアプリケーションに組み込むことが可能です。私はこれらの機能をサポートするモジュール（[indexnow-submitter の発表][18] を参照）も作成しましたので、これを Node ベースのアプリケーションに簡単に組み込むことができます。
```

この記事では、Next.js アプリケーションに `IndexNow` プロトコルを追加する方法について学びました。このプロトコルを活用することで、ウェブサイトに変更を加えるたびに、ページのインデックス作成を自動化かつ迅速に行うことができ、検索エンジンが最新のコンテンツを取得できるようになります。

この記事が役立つことを願っています。さらにカスタマイズしたり、自分に合った統合方法を自由に試してみてください。

## リンクと参考資料

- [IndexNow ドキュメント][19]
- [FAQ][20]
- [IndexNow Bing][21]
- [indexnow-submitter NPM モジュール][22]
- [indexnow-submitter リリース発表][23]

---

![Vivek Sahu](https://www.freecodecamp.org/news/content/images/size/w60/2024/08/avatar.jpeg)

BITS Pilani の CSE 卒業生で、~8 年間にわたり MNC と製品ベースのスタートアップ（HR SAAS、eコマースウェブアプリ、ゲーム、クラウドインフラ）で多岐にわたる経験を積んできました。フリーランスとして https://wenixtech.com でも活動中です。

---

ここまで読んでくれてありがとう。感謝の気持ちを伝えましょう。ありがとうと言ってみてください。

無料でコーディングを学びましょう。freeCodeCamp のオープンソースカリキュラムは、40,000 人以上の人々が開発者として就職するのを助けています。[今すぐ開始][24]

[1]: #what-is-indexnow
[2]: #how-does-it-work
[3]: #steps
[4]: #prerequisites
[5]: #high-level-steps
[6]: #how-to-prove-ownership-of-the-host
[7]: #how-to-create-a-script-to-get-all-urls-from-your-sitemap
[8]: #how-to-call-the-indexnow-api
[9]: #next-steps-and-improvements
[10]: #conclusion
[11]: #links-and-references
[12]: https://www.indexnow.org/
[13]: https://www.indexnow.org/
[14]: https://www.bing.com/indexnow/getstarted
[15]: https://www.bing.com/indexnow/getstarted
[16]: https://www.bing.com/indexnow/getstarted#implementation
[17]: https://localhost:3000/f34f184d10c049ef99aa7637cdc4ef04.txt
[18]: https://wewake.dev/posts/introducing-indexnow-submitter-fast-search-engine-indexing/
[19]: https://www.indexnow.org/documentation
[20]: https://www.indexnow.org/faq
[21]: https://www.bing.com/indexnow/getstarted#implementation
[22]: https://www.npmjs.com/package/indexnow-submitter
[23]: https://wewake.dev/posts/introducing-indexnow-submitter-fast-search-engine-indexing/
[24]: https://www.freecodecamp.org/learn/

