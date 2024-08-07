---
title: Next.js アプリを素早くインデックスする方法 — IndexNow の活用
date: 2024-08-07T16:16:24.360Z
author: Vivek Sahu
authorURL: https://www.freecodecamp.org/news/author/wewake/
originalURL: https://www.freecodecamp.org/news/how-to-index-nextjs-pages-with-indexnow/
translator: ""
reviewer: ""
---

Next.js は超高速なアプリケーションを構築するための強力なフレームワークです。しかし、これらのアプリケーションが検索エンジンに迅速にインデックスされることは、可視性とトラフィックの面で非常に重要ですが、残念ながらこれがすぐには実現しません。

<!-- more -->

サイトマップをアップロードした後でも、検索エンジンがページをクロールするまでに数週間から数ヶ月かかることがあります。特に、新しいページを追加または更新した場合、検索エンジンがそれに気付くまでに数週間かかることがあるのです。

では、もっと効率的な方法はないのでしょうか？

この記事では、IndexNow を活用して、Bing や Yahoo などの主要な検索エンジンで Next.js アプリの SEO パフォーマンスを向上させる方法を紹介します。

## 目次

-   [IndexNow とは？][1]
    -   [動作の仕組み][2]
-   [手順][3]
    -   [前提条件][4]
    -   [全体の流れ][5]
    -   [ホストの所有権の証明方法][6]
    -   [サイトマップからすべての URL を取得するスクリプトの作成方法][7]
    -   [IndexNow API を呼び出す方法][8]
-   [次のステップと改善点][9]
-   [結論][10]
-   [リンクと参考文献][11]

## IndexNow とは？

[IndexNow][12] は、インデックスの時間を大幅に短縮するプロトコルです。公式サイトでは以下のように定義されています：

> IndexNow は、ウェブサイトの最新のコンテンツ変更を検索エンジンに即座に通知する簡単な方法です。最もシンプルな形では、IndexNow は検索エンジンに URL やその内容が追加、更新、または削除されたことを知らせるためのシンプルなピンであり、検索エンジンがこの変更を検索結果に迅速に反映できるようになります。([出典: IndexNow ホーム][13])

このプロトコルは Bing、Naver、Seznam.cz、Yandex、Yep などが採用しています。この記事を書いている時点では、Google はこのプロトコルをサポートしていません。

Wix などの多くの CMS にネイティブに統合されており、Drupal や WordPress 向けのサードパーティプラグインも多数存在します。ただし、Next.js にはネイティブサポートがありません。

### 動作の仕組み

何かを更新するたびに、この API を「ping」して変更を通知するだけです。

これにより検索エンジンは、この情報を受け取り、他の自然にクロールされている URL よりも優先的にこれらの URL をクロールすることができます。

このガイドでは、Next.js アプリに IndexNow を統合し、URL の変更が検索エンジンに素早く通知され、インデックスされるようにする手順を説明します。

### 前提条件

-   Next.js アプリ。
-   Next.js アプリの **sitemap.xml**。

### 全体の流れ

1.  URL を送信するホストの「所有権の証明」が必要です。
2.  サイトマップからすべての URL を取得するシンプルな Node.js スクリプトを作成します。
3.  IndexNow API を呼び出します。

### ホストの所有権の証明方法

Bing の [`IndexNow`][14] [ページ][15] にアクセスします。Next.js 向けの直接的な統合はないため、[手動での統合][16] セクションまでスクロールします。

「Generate」をクリックして新しい API キーを生成します。

Next.js アプリでプロジェクトのルートにある **public** ディレクトリに移動します。すべての静的コンテンツはこのディレクトリを通してレンダリングされます。新しいファイルを作成し、この API キーを保存します：

```bash
# API キーが "f34f184d10c049ef99aa7637cdc4ef04" と仮定します。生成された API キーに応じて変更してください
echo "f34f184d10c049ef99aa7637cdc4ef04" > f34f184d10c049ef99aa7637cdc4ef04.txt
```

Next.js アプリをビルドして実行します：

```bash
npm run build && npm run start
```

その後、ファイルが `/f34f184d10c049ef99aa7637cdc4ef04.txt` パスで利用できるか確認します。

つまり、[https://localhost:3000/f34f184d10c049ef99aa7637cdc4ef04.txt][17] を開くと、ブラウザに「f34f184d10c049ef99aa7637cdc4ef04」というテキストが表示されるはずです。

API キーに応じて上記の値を変更してください。変更をコミットしてプッシュし、これを本番環境にデプロイします。

デプロイが成功したら、`<Your URL>/<API Key>.txt` が `<API Key>` のテキストをレンダリングすることを確認します。つまり：`<Your URL>/f34f184d10c049ef99aa7637cdc4ef04.txt` が `f34f184d10c049ef99aa7637cdc4ef04` をレンダリングする必要があります。

### サイトマップからすべての URL を取得するスクリプトの作成方法

まず、Node スクリプトファイルを作成します：

```bash
touch lib/indexnow.js
```

次に、以下のコードを追加します：

```js
const xml2js = require('xml2js');

// 設定
const sitemapUrl = '<Your URL>/sitemap.xml'; // TODO: 更新する
const host = '<Your URL>'; // TODO: 更新する
const key = '<API Key>'; // TODO: 更新する
const keyLocation = 'https://<Your URL>/<API Key>.txt'; // TODO: 更新する

const modifiedSinceDate = new Date(process.argv[2] || '1970-01-01');

if (isNaN(modifiedSinceDate.getTime())) {
  console.error('無効な日付が提供されました。フォーマット YYYY-MM-DD を使用してください');
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

基本的に、指定された日時以降に更新された URL をサイトマップから取得しています。日時はスクリプトへの引数として渡すことができます。

次に、サイトマップからの XML レスポンスを解析するために `xml2js` ライブラリをインストールしてください。

```bash
npm install xml2js --save-dev
```

その後、スクリプトを実行して URL を取得し、すべてが正常に動作するか確認します：

```bash
node lib/indexnow.js 2024-01-01
```

これにより、2024 年 1 月 1 日以降に更新された URL のリストが出力されます。任意の日付を渡すことができます。

### IndexNow API の呼び出し方法

以下が IndexNow API のスキーマです：

**リクエスト：**

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

**レスポンス：**

| HTTP コード | レスポンス | 理由 |
| --- | --- | --- |
| 200 | Ok | URL が正常に送信された |
| 400 | Bad request | 不正なフォーマット |
| 403 | Forbidden | キーが無効な場合（例：キーが見つからない、キーがファイルに含まれていない） |
| 422 | Unprocessable Entity | URL がホストに属していない、またはプロトコルのスキーマにキーが一致しない場合 |
| 429 | Too Many Requests | リクエストが多すぎる（スパムの可能性） |

URL 取得部分が正しく動作することを確認したので、IndexNow API を呼び出すメイン関数を追加しましょう。

お気に入りの IDE で `lib/index.js` ファイルを開き、次の関数を追加します：

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

この関数は、URL リストと `<API Key>` を渡して IndexNow API を呼び出します。

この関数をメイン関数から呼び出します。メイン関数を次のように修正します：

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
      console.log('No URLs modified since the specified date.');
    }
  } catch (error) {
    console.error('An error occurred:', error);
  }
}
```

これで、指定した URL に対して IndexNow API が呼び出されるようになりました。

スクリプトを実行すると、成功時には次のような出力が表示されたり、エラーがある場合にはエラーメッセージが表示されたりします：

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

これで、あなたの API が IndexNow を使って迅速なインデックス付けを行うことができます。

## 次のステップと改善点

この記事では、IndexNow を介してローカルでスクリプトを作成し実行する方法を見てきました。しかし、さらに改善する方法がいくつかあります。例えば：

-   自動更新のために IndexNow を CI/CD パイプラインに統合すること。
-   大きなサイトマップを効率的に処理するためにバッチ処理やキューを使用すること。
-   インデックス送信のモニタリングとログを行い、デバッグや分析に役立てること。
-   IndexNow API の追加機能（例えば URL の削除）を探ること。
-   CLI 版を提供すること。
-   TypeScript のサポートを追加すること。

これらはこの記事の範疇を超えていますが、これらの高度な機能の一部またはすべてを実装した npm モジュールがいくつか存在し、Node ベースのアプリケーションに簡単に統合できます。例えば、私が作成したもの（[indexnow-submitter 発表を参照][18]）もありますし、エコシステムにおける欠落した機能やサポートを追加するものもあります。これらのモジュールをあなたの Node アプリケーションにプラグインして使用することができます。
```

この記事では、Next.js アプリケーションに `IndexNow` プロトコルを追加する方法を学びました。このプロトコルを活用することで、ウェブサイトに変更を加えるたびに、より迅速で自動化された便利なページインデックス体験が可能になります。これにより、検索エンジンがページの最新コンテンツを取得できるようになります。

この記事が役立ったと感じていただけたなら、ぜひ更なる実験やカスタマイズをして、ご自身のニーズに合わせてみてください。

## リンクと参考資料

- [IndexNow Documentation][19]
- [FAQ][20]
- [IndexNow Bing][21]
- [indexnow-submitter NPM モジュール][22]
- [indexnow-submitter リリースアナウンス][23]

---

![Vivek Sahu](https://www.freecodecamp.org/news/content/images/size/w60/2024/08/avatar.jpeg)

BITS Pilani 卒のコンピュータサイエンス学士。MNC やプロダクトベースのスタートアップ（HR SAAS、Eコマースウェブアプリ、ゲーム、クラウドインフラ）で約 8 年間の多様な経験を持つフリーランサー。https://wenixtech.com で活動中。

---

ここまで読んでくださった方、感謝の意を表して「Say Thanks」をしましょう。

無料でコーディングを学びましょう。freeCodeCamp のオープンソースカリキュラムは、40,000 人以上の人々が開発者としての仕事を得る手助けをしています。[今すぐ始める][24]

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

