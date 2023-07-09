<!--
title:   XRPLのオーダーブックを使ってみよう
tags:    Blockchain,Web3,XRPLedger,xrp
id:      bc33b2c370d861969ed3
private: false
-->
XRP LedgerのDEX（分散型取引所）では、ユーザーが自分のデジタル資産をオファーとして提供することができます。オファーは、他のユーザがその資産を購入するために使用できます。

オファーには、取引金額、支払い通貨、受け取り通貨などの情報が含まれます。オファーは自由に設定でき、ユーザーは自分が望む価格でアセットを販売することができます。

オファーはXRP Ledger上で公開されており、個別のオファーだけでなくペアごとのオファーを一括で取得することも可能です。

今回はそのオファーデータを取得し、オーダーブック形式で表示する方法を解説します。

https://xrpl.org/decentralized-exchange.html

## 利用例

オーダーブック機能を利用し表示しているプロジェクトの1つとしてSologenicが挙げられます。
このプロジェクトはオーダーブックの表示だけではなく、チャートの表示、売買機能も搭載しています。

![image](https://user-images.githubusercontent.com/69445828/221449804-a8bc68c7-95ec-419a-915c-8abe9310efe0.png)

## 準備

今回はSavaScriptのクライアントライブラリであるxrpl.jsを利用します。

https://github.com/XRPLF/xrpl.js

https://www.npmjs.com/package/xrpl

https://js.xrpl.org/

JavaScript以外にもPythonなどの他の言語のライブラリも存在します。
チェックしてみてください。

https://xrpl.org/ja/client-libraries.html

オーダーブックを取得するには

- レジャーデータからオファーオブジェクトを直接取得する
- `book_offers`メソッドでask,bidのオーダーブックをそれぞれ取得する
- `subscribe`メソッドでask,bidのオーダーブックを一括取得する

など、いくつかの方法がありますが、今回は`subscribe`メソッドを使ってみたいと思います。

## subscribeメソッド

subscribeメソッドはXRPLで特定のイベントが発生した場合に、メッセージを通知するようにサーバーへ要求するコマンドです。

利用可能なイベントとして、レジャーの検証やトランザクション、アカウントのトランザクションなどが存在します。

ここではオーダーブックイベントを利用し、データを取得します。

https://xrpl.org/ja/subscribe.html#subscribe

## 注文情報を取得する

以下のコードでペアの注文情報を取得することができます。
ここではメインネットのXRP/USD.rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59Bペアを利用します。

```js
const client = new Client('wss://xrpl.ws');
await client.connect()
const response = await client.request({
  command: 'subscribe',
  books: [
    {
      taker_pays: { currency: 'XRP' },
      taker_gets: { issuer: 'rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B', currency: 'USD' },
      snapshot: true,
      both: true,
    },
  ],
});
console.log(response.result);
```

コード内の`snapshot`フィールドがtrueの場合、レスポンスにリクエスト時点での注文情報を含めるようになります。
`both`フィールドがtrueの場合、XRP→USDの注文情報だけでなくUSD→XRPと逆サイドの注文も含めるようになります。

このコードの実行結果は以下のようになります。

```js
{
  asks: {
    {
      Account: 'rBndiPPKs9k5rjBb7HsEiqXKrz8AfUnqWq',
      BookDirectory: 'DFA3B6DDAB58C7E8E5D944E736DA4B7046C30E4F460FD9DE4E0D6ACC35757000',
      BookNode: '0',
      Flags: 0,
      LedgerEntryType: 'Offer',
      OwnerNode: '0',
      PreviousTxnID: '7B88A6D13F0CEF15EF28649C3C7A3E6B987B23B36E1ED04AD8F13839B7F7C3ED',
      PreviousTxnLgrSeq: 78070216,
      Sequence: 2430764,
      TakerGets: '680762087',
      TakerPays: {
        currency: 'USD',
        issuer: 'rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B',
        value: '257.0966105653182'
      },
      index: '43FF075A07BCCE53091C436F3FB428BED2F17501769230397120DB8930BF7B1C',
      owner_funds: '3206390705',
      quality: '0.00000037766'
    },
    {
      Account: 'rPbMHxs7vy5t6e19tYfqG7XJ6Fog8EPZLk',
      BookDirectory: 'DFA3B6DDAB58C7E8E5D944E736DA4B7046C30E4F460FD9DE4E0D6F21B778F500',
      BookNode: '0',
      Flags: 0,
      LedgerEntryType: 'Offer',
      OwnerNode: '0',
      PreviousTxnID: 'B44C3E00D72405D87B1DFE6BF968E905653F0CAE04803BF78F462E4D80430732',
      PreviousTxnLgrSeq: 78072721,
      Sequence: 1070303,
      TakerGets: '2000000000',
      TakerPays: {
        currency: 'USD',
        issuer: 'rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B',
        value: '756.27306'
      },
      index: '27D4658890B561C40ED8820FE61338D5FC6727C39509EC1C24C57F5189D84B20',
      owner_funds: '5375920030',
      quality: '0.00000037813653'
    }
    ...
  },
  bids: {
    {
      Account: 'rsgi7ENscrVaXC44JE2m94XzKQrmAVX2gV',
      BookDirectory: '4627DFFCFF8B5A265EDBD8AE8C14A52325DBFEDAF4F5C32E5B096C6EFAA91865',
      BookNode: '0',
      Expiration: 762310783,
      Flags: 0,
      LedgerEntryType: 'Offer',
      OwnerNode: '0',
      PreviousTxnID: 'BF475896247E2FCABB8FEF3C440F2376E5B6A793B2AF0B14C6B7F18DB3D4AD6E',
      PreviousTxnLgrSeq: 78070492,
      Sequence: 72701643,
      TakerGets: {
        currency: 'USD',
        issuer: 'rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B',
        value: '20.41489722360511'
      },
      TakerPays: '54150487',
      index: 'EC40BD275BF22AA9C40F024EDF6A642ABD3D73A4AA8CD7313330C45C6C4DA6EA',
      owner_funds: '20.37571359826066',
      quality: '2652498.697984101',
      taker_gets_funded: {
        currency: 'USD',
        issuer: 'rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B',
        value: '20.37571359826066'
      },
      taker_pays_funded: '54046553'
    }
    {
      Account: 'rnW98SKRCuw7RM2HDWgNfZJ8qPVu1tUBaU',
      BookDirectory: '4627DFFCFF8B5A265EDBD8AE8C14A52325DBFEDAF4F5C32E5B096C6F97BACA6A',
      BookNode: '0',
      Expiration: 762310774,
      Flags: 0,
      LedgerEntryType: 'Offer',
      OwnerNode: '0',
      PreviousTxnID: '05DB68CCE660DF867A9A7380F8FCB39FD83129FBA095A9C2C1D6B3E6BE2CCCCD',
      PreviousTxnLgrSeq: 78069211,
      Sequence: 68708233,
      TakerGets: {
        currency: 'USD',
        issuer: 'rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B',
        value: '49.161144'
      },
      TakerPays: '130400000',
      index: '72D8A1605F35F422740B30C9FA823392DFCC3D77C3E5D4BFF9EB9730CB54A9EE',
      owner_funds: '49.122187',
      quality: '2652501.333166698',
      taker_gets_funded: {
        currency: 'USD',
        issuer: 'rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B',
        value: '49.122187'
      },
      taker_pays_funded: '130296666'
    },
    ...,
  }
}
```

## オーダーブックの作成

これらの注文情報からオーダーブックを作っていきましょう。
現在bidsとasksにはOfferオブジェクトが複数格納されています。

https://xrpl.org/offer.html#offer

このOfferオブジェクトから価格と注文量を取得します。
ここではXRP/USDのオーダーブックとし、価格は XRP/USD、注文量はXRP建てで表示することとします。

### 価格を揃える

asksとbidsのqualityにはXRP/USDとUSD/XRPの価格情報が設定されています。
またXRPはdropでの価格(1XRP=1,000,000)になっているため、XRPでの表示とします。

これらを揃えるために、asks内のquality: '0.00000037766'は、1000000を掛けることで0.37766となります。

また、bids内のquality: '2652498.697984101'は、1000000をかけることで0.37700となります。

javascriptでの少数計算での誤差をなくすために、**bignumber.js**を使います。

```js
import BigNumber from 'bignumber.js';

const asksPrice = response.result.asks.map((ask) => {
  return BigNumber(ask.quality).multipliedBy(1000000).toString()
})

const bidsPrice = response.result.bids.map((ask) => {
  return BigNumber(1).dividedBy(BigNumber(ask.quality)).multipliedBy(1000000).toString()
})

console.log(asksPrice)
console.log(bidsPrice)
```

実行結果は以下のようになります。

```js
[
  '0.37766',            '0.37831724',         '0.3790894153652532',
  '0.379669541',        '0.3796787613004684', '0.3819853403971212',
  '0.38566128506197',   '0.386',              '0.38818016915949',
  '0.3885826132620423', '0.39',               '0.3900294515983404',
  '0.3905376246907444', '0.3915',             '0.393',
  '0.3933',             '0.3954537361267731', '0.3954545353125844',
  '0.397212',           '0.4',                '0.4048582995951417',
  '0.4090191613927092', '0.41',               '0.41',
  '0.41',               '0.41',               '0.4109073602091388',
  '0.4118107317876703', '0.4155',             '0.4166',
  '0.4199999868148413', '0.420000001',        '0.420333943711323',
  '0.4223054992888399', '0.4275392890329845', '0.4299',
  '0.4309977911012938', '0.431',              '0.4333',
  '0.44',               '0.44',               '0.443',
  '0.4433',             '0.445',              '0.449',
  '0.45',               '0.45',               '0.45',
  '0.45',               '0.45',               '0.45',
  '0.4533',             '0.4533',             '0.455',
  '0.459',              '0.46',               '0.468',
  '0.4683995391705069', '0.48',               '0.48'
]
[
  '0.37700301257829', '0.37700263803681', '0.377',
  '0.37264198009708', '0.37254567512024', '0.369993133',
  '0.36999155268118', '0.36887390482392', '0.36600365988429',
  '0.366',            '0.365',            '0.365',
  '0.36496350364964', '0.3645',           '0.364',
  '0.36363636363636', '0.361',            '0.36',
  '0.36',             '0.36',             '0.355',
  '0.355',            '0.3533',           '0.35158933333333',
  '0.3515',           '0.351',            '0.35099999',
  '0.35000020162269', '0.35',             '0.35',
  '0.35',             '0.35',             '0.347212',
  '0.347',            '0.342',            '0.342',
  '0.342',            '0.34',             '0.337',
  '0.337',            '0.336',            '0.33558933333333',
  '0.335',            '0.3336',           '0.33155',
  '0.3311',           '0.331',            '0.33',
  '0.33',             '0.3259',           '0.32199702534729',
  '0.32',             '0.31858928571429', '0.316',
  '0.316',            '0.31501000000283', '0.315',
  '0.3134',           '0.31310001',       '0.313'
]
```

## 注文量の取得

こちらは価格とは異なり、オファー内の情報を取得し、XRP量へ変換するだけになります。

```js
const asksAmount = response.result.asks.map((ask) => {
  return BigNumber(ask.TakerGets).dividedBy(1000000).toString()
})

const bidsAmount = response.result.bids.map((ask) => {
  return BigNumber(ask.TakerPays).dividedBy(1000000).toString()
})

console.log(asksAmount)
console.log(bidsAmount)
```

```js
[
  '680.762087', '2000',        '38000',       '4659.393721',
  '12.223249',  '5980.570901', '3.92448',     '5',
  '300',        '5.23675',     '20',          '2.79102',
  '2.65557',    '500',         '5',           '600',
  '4.71772',    '4.48183',     '7500',        '7729',
  '247',        '4.68181',     '1489.645582', '1007',
  '10',         '4000',        '2.89951',     '53.290236',
  '690.206805', '10000',       '1736.801245', '100',
  '6.82091',    '4.77389',     '7.37165',     '233',
  '7.9225',     '10000',       '1000',        '2000',
  '10',         '3000',        '1000',        '650',
  '3000',       '5000',        '500',         '500',
  '2500',       '5000',        '0.1',         '1000',
  '233',        '116',         '1000',        '4470.444247',
  '1000',       '217',         '1000',        '391'
]
[
  '54.150487',    '130.4',        '13655.346237', '1303.871344',
  '200000',       '6000',         '0.000002',     '2975.028395',
  '1202.834003',  '9219.166772',  '200',          '5',
  '19.2074',      '6900',         '414.622995',   '58.575',
  '1500',         '390',          '55813.635195', '100000',
  '10.5',         '623.591549',   '56.33',        '7500',
  '1898',         '2000',         '3647',         '1.708102',
  '10076.240904', '0.01',         '5',            '657771.42',
  '7500',         '43',           '900',          '500',
  '2550',         '48369.695428', '400',          '200',
  '297.6193',     '7500',         '10.5',         '225',
  '6600',         '100',          '1',            '0.01',
  '449',          '10',           '228.160191',   '12.735193',
  '5600',         '2531.64',      '31.6456',      '4457.001365',
  '440',          '2000',         '50000',        '100'
]
```

### オーダーブックの表示

価格情報と注文量情報を合わせてオーダーブック情報を表示しましょう。

表示する際、asksは最良レートは要素番号が小さい方から格納されているのでreverse()メソッドで要素を反転させます。

```js
const asks = response.result.asks.map((ask) => {
  return {
    price: BigNumber(ask.quality).multipliedBy(1000000).toString(),
    amount: BigNumber(ask.TakerGets).dividedBy(1000000).toString()
  }
})

const bids = response.result.bids.map((ask) => {
  return {
    price: BigNumber(1).dividedBy(BigNumber(ask.quality)).multipliedBy(1000000).toString(),
    amount: BigNumber(ask.TakerPays).dividedBy(1000000).toString()
  }
})

asks.reverse().forEach(ask => {

  console.log(((ask.amount + Array(25).join(' ')).slice(0, 24)) + (ask.price + Array(20).join(' ')).slice(0, 20))
})
console.log('===========================================================')
bids.forEach(bid => {
  console.log(Array(25).join(' ') + (bid.price + Array(20).join(' ')).slice(0, 20) + bid.amount)
})
```

```sh
...
20                      0.39
5.23675                 0.3885826132620423
300                     0.38818016915949
5                       0.386
3.92448                 0.38566128506197
4676.707211             0.3796489755481657
5985.802251             0.3794550839673051
38000                   0.379446704
12.223249               0.3790894153652532
1895.294448             0.3784276799981424
=====================================================
                        0.37700263803681    36.076796
                        0.377               13655.346237
                        0.372861987         200000
                        0.37285991744149    2975.028395
                        0.37273139          2000
                        0.37254567512024    0.000002
                        0.36713637817091    6000
                        0.36600365988429    1202.834003
                        0.366               9219.166772
                        0.365               200
...
```

### オーダーブックの更新

オーダーブックを取得後にも連続的にDEXに対するトランザクションは送信されます。subscribeメソッド実行後にtransaction streamを取得することで該当のオーダーブックに変更があった場合に変更に関わったトランザクションデータを取得することができます。

複雑な処理となるため、今回は省略します。

```js
client.on('transaction', message => {
  console.log(message)
})
```

## まとめ

第三者が提供しているDEX APIサービスでは、データの提供が突然止まったり不正なデータを提供される可能性がゼロではありません。
XRP LedgerではネイティブなDEX機能により、DEXの注文やオーダーブック情報をノードから直接取得することが可能です。

XRP Ledgerに興味がある方は開発者Discordチャンネルへ是非お越しください！
日本語チャンネルもありますので、英語が出来なくても大丈夫です！

https://discord.gg/aBH9MwcsbR