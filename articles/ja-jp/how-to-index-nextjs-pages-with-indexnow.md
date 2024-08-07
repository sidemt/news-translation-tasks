---
title: 次の Next.js アプリを IndexNow で早くインデックスする方法
date: 2024-08-07T16:11:04.738Z
author: Vivek Sahu
authorURL: https://www.freecodecamp.org/news/author/wewake/
originalURL: https://www.freecodecamp.org/news/how-to-index-nextjs-pages-with-indexnow/
translator: ""
reviewer: ""
---

Next.js は超高速なアプリケーションを構築するための強力なフレームワークです。しかし、検索エンジンに迅速にインデックスされることが、可視性とトラフィックにとって重要です。残念ながら、これは即座には行われません。

<!-- more -->

サイトマップをアップロードした後でも、検索エンジンがページをクロールするまでに数週間から数ヶ月かかることがあります。新しいページを追加したり更新したりした場合、検索エンジンがそれを認識するまで数週間かかることもあります。

では、もっと早くする方法はないのでしょうか？

この記事では、Next.js アプリを IndexNow を使用して Bing や Yahoo などの主要な検索エンジンで迅速にインデックスさせる方法を学びます。

## 目次

-   [IndexNow とは?][1]
    -   [どのように機能するのか?][2]
-   [手順][3]
    -   [前提条件][4]
    -   [高レベルの手順][5]
    -   [ホストの所有権を証明する方法][6]
    -   [サイトマップから URL を取得するスクリプトを作成する方法][7]
    -   [IndexNow API の呼び出し方法][8]
-   [次のステップと改善点][9]
-   [結論][10]
-   [リンクと参考文献][11]

## IndexNow とは?

[IndexNow][12] は、インデックス時間を劇的に短縮するプロトコルです。彼らのウェブサイトでは次のように定義されています。

> IndexNow は、ウェブサイトの所有者がウェブサイトの最新の内容の変更を検索エンジンに即座に通知する簡単な方法です。最もシンプルな形では、IndexNow は URL とその内容が追加、更新、または削除されたことを検索エンジンに知らせる簡単な ping です。これにより、検索エンジンは検索結果でこの変更を迅速に反映することができます。([出典：IndexNow ホーム][13])

Bing、Naver、Seznam.cz、Yandex、Yep などで採用されています。ただし、この記事の執筆時点では Google はこのプロトコルをサポートしていません。

Wix などの多くの CMS にはネイティブ統合されていますし、Drupal や WordPress のような他の CMS 向けのサードパーティプラグインもたくさんあります。ただし、Next.js にはネイティブサポートがありません。

### どのように機能するのか?

何かを更新するたびに、「ping」、つまり API を呼び出してその変更を通知するだけで済みます。

この情報が受信されると、検索エンジンは自然にクロールされる他の URL よりも優先してこれらの URL をクロールするようになります。

このガイドでは、URL の変更を検索エンジンに提出し、インデックスさせるために IndexNow を既存の Next.js アプリに統合するプロセスを紹介します。

### 前提条件

-   Next.js アプリが必要です。
-   Next.js アプリ用の **sitemap.xml** が必要です。

### 高レベルの手順

1.  最初に、URL を送信するホストの所有権を「証明」する必要があります。
2.  サイトマップからすべての URL を取得する簡単な Node.js スクリプトを作成します。
3.  IndexNow API を呼び出します。

### ホストの所有権を証明する方法

Bing の [`IndexNow`][14] [ページ][15]に移動します。Next.js 用の直接的な統合はないので、ページをスクロールして [手動統合][16] セクションを探します。

「Generate」をクリックして新しい API キーを生成します。

Next.js アプリの **public** ディレクトリに移動します。すべての静的コンテンツはこのディレクトリ経由でレンダリングされます。新しいファイルを作成し、この API キーを保存します。

```bash
# API キーが "f34f184d10c049ef99aa7637cdc4ef04" だと仮定します。生成された API キーに応じて変更してください
echo "f34f184d10c049ef99aa7637cdc4ef04" > f34f184d10c049ef99aa7637cdc4ef04.txt
```

Next.js アプリをビルドして実行します。

```bash
npm run build && npm run start
```

その後、パス `/f34f184d10c049ef99aa7637cdc4ef04.txt` にファイルがあることを確認します。

つまり、[https://localhost:3000/f34f184d10c049ef99aa7637cdc4ef04.txt][17] を開くと、ブラウザに「f34f184d10c049ef99aa7637cdc4ef04」というテキストが表示されるはずです。

API キーに応じて上記のキー値を変更してください。この変更をコミットしてプッシュし、本番環境にデプロイします。

デプロイが成功したら、`<Your URL>/<API Key>.txt` が `<API Key>` テキストをレンダリングすることを確認します。つまり、`<Your URL>/f34f184d10c049ef99aa7637cdc4ef04.txt` は `f34f184d10c049ef99aa7637cdc4ef04` をレンダリングするはずです。

### サイトマップから URL を取得するスクリプトを作成する方法

まず、Node スクリプトファイルを作成します。

```bash
touch lib/indexnow.js
```

その後、以下のコードを追加します。

```js
const xml2js = require('xml2js');

// Configuration
const sitemapUrl = '<Your URL>/sitemap.xml'; // TODO: Update
const host = '<Your URL>'; // TODO: Update
const key = '<API Key>'; // TODO: Update
const keyLocation = 'https://<Your URL>/<API Key>.txt'; // TODO: Update

const modifiedSinceDate = new Date(process.argv[2] || '1970-01-01');

if (isNaN(modifiedSinceDate.getTime())) {
  console.error('Invalid date provided. Please use format YYYY-MM-DD');
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


```markdown
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

このスクリプトでは、指定した日付以降に更新されたサイトマップの URL を取得します。日付はスクリプトの引数として渡すことができます。

次に、サイトマップの XML レスポンスを解析するために `xml2js` ライブラリをインストールします。

```bash
npm install xml2js --save-dev
```

その後、スクリプトを実行して URL を取得し、正しく動作するか確認しましょう：

```bash
node lib/indexnow.js 2024-01-01
```

これにより、2024 年 1 月 1 日以降に更新された URL のリストが出力されます。任意の日付を渡すことができます。

### IndexNow API を呼び出す方法

以下に IndexNow API のスキーマを示します：

**リクエスト:**

```
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

| HTTP Code | Response | Reasons |
| --- | --- | --- |
| 200 | Ok | URL submitted successfully |
| 400 | Bad request | Invalid format |
| 403 | Forbidden | In case of key not valid (e.g., key not found, file found but key not in the file) |
| 422 | Unprocessable Entity | In case URLs don't belong to the host or the key is not matching the schema in the protocol |
| 429 | Too Many Requests | Too Many Requests (potential Spam) |

URL 取得部分が正しく動作することを確認したら、IndexNow API を呼び出すためのメイン機能を追加しましょう。

お気に入りの IDE を使用して `lib/index.js` ファイルを開き、以下の関数を追加します：

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

この関数は、渡された URL リストを含むデータを IndexNow API に送信します。

この関数をメイン機能から呼び出すようにします。メイン関数を以下のように変更します：

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

これで、指定した URL ごとに IndexNow API が呼び出されます。

スクリプトを実行すると、成功時には次のような出力が表示されます。問題が発生した場合はエラーメッセージが表示されます：

```
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

お見事！これで、あなたの API は IndexNow を使用して迅速にインデックスされるようになります。

## 次のステップと改善点

この記事では、IndexNow を使用してページをローカルでインデックスするスクリプトの作成と実行方法について説明しました。しかし、さらに改善するために以下の項目を考慮することができます：

-   自動更新のために CI/CD パイプラインに IndexNow を統合する。
-   バッチ処理やキューを使用して大規模なサイトマップを効率的に処理する。
-   デバッグや分析のために IndexNow の送信を監視しログを記録する。
-   IndexNow API の追加機能（例えば URL 削除）を探索する。
-   CLI バージョンを提供する。
-   TypeScript サポートを追加する。

これらはこの記事の範囲を超えますが、このような高度な機能を実装したプロダクションレディな npm モジュールがいくつか存在し、それらをアプリに統合することができます。私は一部の機能やサポートを追加するために [indexnow-submitter announcement][18] を作成しました。Node ベースのアプリケーションにこれらのモジュールを簡単にプラグインして使用することができます。
```

この記事では、Next.js アプリケーションに `IndexNow` プロトコルを追加する方法について説明しました。このプロトコルを活用することで、ウェブサイトに変更を加えた際、検索エンジンが最新の内容を迅速かつ自動的にインデックス化できるようになり、ページのインデックス化体験を格段に向上させることが可能です。

この記事が役に立ったことを願っています。ぜひ、この統合をさらにカスタマイズして、あなたのニーズに合った形に仕上げてみてください。

## リンクと参考情報

-   [IndexNow ドキュメント][19]
-   [FAQ][20]
-   [IndexNow Bing][21]
-   [indexnow-submitter NPM モジュール][22]
-   [indexnow-submitter リリース発表][23]

---

![Vivek Sahu](https://www.freecodecamp.org/news/content/images/size/w60/2024/08/avatar.jpeg)

BITS Pilani 出身の CSE 卒業生で、約8年にわたり MNC とプロダクトベースのスタートアップ (HR SAAS、e-commerce ウェブアプリ、ゲーム、クラウドインフラ) での多岐にわたる経験を持つ。現在はフリーランスとして https://wenixtech.com で活動中。

---

ここまで読んでくれてありがとう。ぜひ、作者に感謝の気持ちを伝えてください。お礼を言う

無料でコーディングを学びましょう。freeCodeCamp のオープンソースカリキュラムは、40,000 人以上がデベロッパーとして就職するのを助けてきました。[今すぐ始める][24]

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

