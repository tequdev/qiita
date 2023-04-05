<!--
title:   秘密鍵紛失時のアカウントリカバリー【XRP Ledger】
tags:    Blockchain,XRPLedger,web3,xrp
id:      ca45999dec66e242a8a2
private: false
-->
## はじめに

XRP Ledgerは、分散型台帳技術を採用したブロックチェーンの1つです。2012年にXRP Ledgerは3人の開発者によって開発され、現在もXRPL財団やRipple社、XRPL Labsなどを始めとしたXRPLコミュニティによって開発が続けられています。

暗号資産を利用する上で秘密鍵は非常に重要な役割を担っています。秘密鍵を紛失した場合、一般的にはアカウントの資産を取り戻すことはできません。本記事では、XRP Ledgerにおける秘密鍵紛失時でもアカウントの資産にアクセスする方法について解説します。

アカウントリカバリーの方法として次の2つがあります。

- **レギュラーキーを利用したアカウントリカバリー**
- **マルチシグを利用したアカウントリカバリー**

## レギュラーキーを使用したアカウントリカバリー

### レギュラーキーとは

レギュラーキーは、XRP Ledgerのアカウントのマスターキーとは別に設定できるキーです。マスターキーと同様トランザクションへの署名が可能であるため、マスターキーを紛失した場合でもアカウントへのアクセス権を持つことができます。

https://xrpl.org/ja/cryptographic-keys.html#レギュラーキーペア

マスターキーが流出した場合は、マスターキーを無効化することでその後レギュラーキーを使いアカウントを利用し続けることも可能です

### レギュラーキーの設定

https://xrpl.org/ja/assign-a-regular-key-pair.html#レギュラーキーペアの割り当て

https://xrpl.org/ja/setregularkey.html

レギュラーキーは`SetRegularKey`トランザクションにより設定することが可能です。初めてレギュラーキーを設定する場合は、マスターキーが必要となります。

まずはレギュラーキーとするアカウントを作成しましょう。

```js
const { Wallet } = require('xrpl');

const wallet = Wallet.generate();

console.log(wallet);
```
https://stackblitz.com/edit/xrpl-regularkey?file=generate-address.js

実行すると以下のような結果が得られます。
今回はデモのため、値を表示していますが、実際にはprivateKeyやseedが厳重に管理される必要があります。

```sh
Wallet {
  publicKey: 'ED531B7E785F5537CB7BA62B74237519008CEEEC85E864A8F588067AFF59957F9E',
  privateKey: 'ED255BD3ACA7339FE4F669ECE0CE1ED902D761C684E18C1710E49D761A3D3B3081',
  classicAddress: 'rncAosMLDnD2QwMnPxPLqs3ihzPRzSWY3t',
  seed: 'sEdV5mUW2vi9wbFeNvJtBcCmZ23bmu3'
}
```

その後自身のアカウントにレギュラーキーを設定します。

```js
const { Client, Wallet } = require('xrpl');

const client = new Client('wss://testnet.xrpl-labs.com');
// 自身のアカウントのマスターキー(シード)
const wallet = Wallet.fromSeed('sEdTMf58AqhPoKxQ3PK6RBzfN6HYztC');

const main = async () => {
  await client.connect();

  const response = await client.submitAndWait(
    {
      TransactionType: 'SetRegularKey',
      Account: 'rakMd7KDSpnVjgbtfTsCtTu4h7gqJ42DZL',
      // レギュラーキーとするアカウントのアドレス
      RegularKey: 'rncAosMLDnD2QwMnPxPLqs3ihzPRzSWY3t',
    },
    { wallet }
  );
  console.log(response.result.hash);
  // D740755808517728EEB944F60534E43FA3F5A1D27005BB8785E38E2010FB7A3E
};

main();
```
https://stackblitz.com/edit/xrpl-regularkey?file=setRegularKey.js

これでレギュラーキーの設定が完了しました。以下がトランザクションの実行結果です。

https://test.bithomp.com/explorer/D740755808517728EEB944F60534E43FA3F5A1D27005BB8785E38E2010FB7A3E

ページ内のRaw TX dataを見るとレギュラーキーが設定されていることが確認できます。
```json
"FinalFields": {
  "Account": "rakMd7KDSpnVjgbtfTsCtTu4h7gqJ42DZL",
  "Balance": "999999988",
  "Flags": 65536,
  "OwnerCount": 0,
  "RegularKey": "rncAosMLDnD2QwMnPxPLqs3ihzPRzSWY3t",
  "Sequence": 36491881
},
```

### レギュラーキーでのトランザクション署名

レギュラーキーを設定したアカウントは、レギュラーキーを使用してトランザクションを署名することができます。

トランザクション送信者の`rakMd7KDSpnVjgbtfTsCtTu4h7gqJ42DZL`のマスターキーは`sEdTMf58AqhPoKxQ3PK6RBzfN6HYztC`ですが、次のコードではレギュラーキとして設定した`rncAosMLDnD2QwMnPxPLqs3ihzPRzSWY3t`のマスターキー`sEdV5mUW2vi9wbFeNvJtBcCmZ23bmu3`で署名を行っています。

```js
const { Client, Wallet } = require('xrpl');

const client = new Client('wss://testnet.xrpl-labs.com');
// レギュラーキーとしたアカウントのシード
const wallet = Wallet.fromSeed('sEdV5mUW2vi9wbFeNvJtBcCmZ23bmu3');

const main = async () => {
  await client.connect();

  const response = await client.submitAndWait(
    {
      TransactionType: 'Payment',
      Account: 'rakMd7KDSpnVjgbtfTsCtTu4h7gqJ42DZL',
      Destination: 'rQQQrUdN1cLdNmxH4dHfKgmX5P4kf3ZrM',
      Amount: '1000000',
    },
    { wallet }
  );
  console.log(response.result.hash);
  //　02063FA297301B67051FF1BC798BA693D7D45A6519FE6A9BDEB89C9068078B03
};

main();
```
https://stackblitz.com/edit/xrpl-regularkey?file=signTransaction.js

以下がトランザクションの実行結果です。
https://test.bithomp.com/explorer/02063FA297301B67051FF1BC798BA693D7D45A6519FE6A9BDEB89C9068078B03

## マルチシグを利用したアカウントリカバリー

### マルチシグとは

マルチシグとは、複数の署名者がトランザクションへ署名することで、トランザクションを承認する仕組みです。

マスターキーを紛失した場合でも、マルチシグに設定されたアドレスによりトランザクションを作成することでアカウントの管理を継続することができます。

XRP Ledgerではマルチシグのアカウントとして最大32のアドレスを設定することができます。署名に必要な定足数も1~32の範囲で設定することができます。

マルチシグを利用することで以下のようなユースケースが考えられます。

- 1 of 1 でレギュラーキーのような利用方法
- 1 of 3 でアカウントの共同管理
- 3 of 3 でアカウントの厳重な管理

https://xrpl.org/ja/multi-signing.html

### マルチシグの設定

今回は2 of 3のマルチシグを設定します。

まずはマルチシグの署名者とするアカウントを3つ作成しましょう。

```js
const { Wallet } = require('xrpl');

console.log(Wallet.generate());
console.log(Wallet.generate());
console.log(Wallet.generate());
```

```sh
Wallet {
  publicKey: 'ED4A6DC6A7D851C0A92B6349B82C7E7B30AFE02E94B4ECE6AF7615F4A1FDCC2469',
  privateKey: 'ED5B1E73ACFAC6D96286A7BC62951D361D81A0DF8B20137B1A9F020DA37E399EFC',
  classicAddress: 'rBg6tv4fRd8mKKvXQoMg4RpDN8JEQU2GL1',
  seed: 'sEd7yF5VcCsv2FDPgunskUuuKvdb8sg'
}
Wallet {
  publicKey: 'EDEB54971089A74CA51426F1376E33E2F772AD006031F2EB1457EF663DE9AF833A',
  privateKey: 'ED02A945B4C7C2A1CCCA42DA702ED075DC283F029F76C6DE7DED8FB9CF0A2234F3',
  classicAddress: 'rBUjxjCmCwKKyP2gJivhgoBtLksRZ4xGQm',
  seed: 'sEd7cx7AaJA1nHrUC1uAHQB9U6cxzFz'
}
Wallet {
  publicKey: 'ED994F8848463CEC8A35B060C74E7CEE1C5CFBD753EDCE453E785BC84DD160564B',
  privateKey: 'ED2F72CE440A0CFEEA1727EB02444EDA2238E5BE07A07C716F494F4E8D59C3389F',
  classicAddress: 'rDtQwpLzo1CBkCzBaey2Kt6Mcyy6a9ULKj',
  seed: 'sEd7KbkRFTuNj2DYao5Wx8LnhVdEawn'
}
```
https://stackblitz.com/edit/node-lsniff?file=generate-wallet.js

その後自身のアカウントにマルチシグを設定します。

```js
const { Client, Wallet } = require('xrpl');

const client = new Client('wss://testnet.xrpl-labs.com');
const wallet = Wallet.fromSeed('sEdStwpYpMAhfC4g5qjA85pxceRyra7');

const main = async () => {
  await client.connect();

  const response = await client.submitAndWait(
    {
      TransactionType: 'SignerListSet',
      Account: 'rnf4XWcJdALZVg3bUyHGikzfYXfUvUg5vH',
      SignerQuorum: 2,
      SignerEntries: [
        {
          SignerEntry: {
            Account: 'rBg6tv4fRd8mKKvXQoMg4RpDN8JEQU2GL1',
            SignerWeight: 1,
          },
        },
        {
          SignerEntry: {
            Account: 'rBUjxjCmCwKKyP2gJivhgoBtLksRZ4xGQm',
            SignerWeight: 1,
          },
        },
        {
          SignerEntry: {
            Account: 'rDtQwpLzo1CBkCzBaey2Kt6Mcyy6a9ULKj',
            SignerWeight: 1,
          },
        },
      ],
    },
    { wallet }
  );

  console.log(response.result.hash);
  // 1A0A86F78A1FBA91624714CB65BF99954D39E3EDEC38CCED6D088DD77F32D0F9
};

main();
```

以下がトランザクションの実行結果です。
https://test.bithomp.com/explorer/1A0A86F78A1FBA91624714CB65BF99954D39E3EDEC38CCED6D088DD77F32D0F9


https://xrpl.org/ja/signerlistset.html#signerlistset

### マルチシグでのトランザクション署名

トランザクション送信者の`rnf4XWcJdALZVg3bUyHGikzfYXfUvUg5vH`のマスターキーは`sEdStwpYpMAhfC4g5qjA85pxceRyra7`ですが、次のコードではマルチシグの署名者としてとして設定した`rBg6tv4fRd8mKKvXQoMg4RpDN8JEQU2GL1`のマスターキー`sEd7yF5VcCsv2FDPgunskUuuKvdb8sg`および、`rBUjxjCmCwKKyP2gJivhgoBtLksRZ4xGQm`のマスターキー`sEd7cx7AaJA1nHrUC1uAHQB9U6cxzFz`の２つで署名を行っています。

```js
const { Client, Wallet, multisign } = require('xrpl');
const { decode } = require('ripple-binary-codec');

const client = new Client('wss://testnet.xrpl-labs.com');

// rBg6tv4fRd8mKKvXQoMg4RpDN8JEQU2GL1
const wallet1 = Wallet.fromSeed('sEd7yF5VcCsv2FDPgunskUuuKvdb8sg');

// rBUjxjCmCwKKyP2gJivhgoBtLksRZ4xGQm
const wallet2 = Wallet.fromSeed('sEd7cx7AaJA1nHrUC1uAHQB9U6cxzFz');

const main = async () => {
  await client.connect();

  const accountResponse = await client.request({
    command: 'account_info',
    account: 'rnf4XWcJdALZVg3bUyHGikzfYXfUvUg5vH',
  });
  const Sequence = accountResponse.result.account_data.Sequence;

  const tx = {
    TransactionType: 'Payment',
    Account: 'rnf4XWcJdALZVg3bUyHGikzfYXfUvUg5vH',
    Destination: 'rQQQrUdN1cLdNmxH4dHfKgmX5P4kf3ZrM',
    Amount: '1000000',
    Sequence,
    Fee: '50',
  };
  // 署名者の１つで署名
  const tx_blob_1 = wallet1.sign(tx, true).tx_blob;
  // 署名者の１つで署名
  const tx_blob_2 = wallet2.sign(tx, true).tx_blob;
  // 署名の結合
  const tx_blob = multisign([tx_blob_1, tx_blob_2]);
  console.log(JSON.stringify(decode(tx_blob), null, '  '));

  await client.request({
    command: 'submit',
    tx_blob,
  });
};

main();
```

以下は署名結合後のトランザクションデータです。Sigersフィールドに署名者のアカウントアドレスと公開鍵、そして署名が含まれています。

```json
{
  "TransactionType": "Payment",
  "Sequence": 36493102,
  "Amount": "1000000",
  "Fee": "50",
  "SigningPubKey": "",
  "Account": "rnf4XWcJdALZVg3bUyHGikzfYXfUvUg5vH",
  "Destination": "rQQQrUdN1cLdNmxH4dHfKgmX5P4kf3ZrM",
  "Signers": [
    {
      "Signer": {
        "SigningPubKey": "EDEB54971089A74CA51426F1376E33E2F772AD006031F2EB1457EF663DE9AF833A",
        "TxnSignature": "C149F41A1F092FF42A5BF8F45AFFC2E80B2DB212E3E0EDB059F746D6C053EBCE0B82A543140B642505F86815DFB4791EC4D1A7BA92D239CD10CDCD3DF21A8F07",
        "Account": "rBUjxjCmCwKKyP2gJivhgoBtLksRZ4xGQm"
      }
    },
    {
      "Signer": {
        "SigningPubKey": "ED4A6DC6A7D851C0A92B6349B82C7E7B30AFE02E94B4ECE6AF7615F4A1FDCC2469",
        "TxnSignature": "6EF4177D41D7A9F28DD8E87733E3CC9F67CA891838FE8DA33B39CB1BBF2C3226AE0860C8E5F0A1EEFB3EBFBB1C1F1EC3A199F2A6C9336FCA8EA6E51F95DA0B0D",
        "Account": "rBg6tv4fRd8mKKvXQoMg4RpDN8JEQU2GL1"
      }
    }
  ]
}
```

以下がトランザクションの実行結果です
https://test.bithomp.com/explorer/53E175D117192FF40B134C3E1282FF8FDE3B2DC008082395D0D0B887DC4F4929

`rnf4XWcJdALZVg3bUyHGikzfYXfUvUg5vH`のマスターキーがなくとも、`rBg6tv4fRd8mKKvXQoMg4RpDN8JEQU2GL1`と`rBUjxjCmCwKKyP2gJivhgoBtLksRZ4xGQm`の２つの署名者によってでトランザクションを実行することができました。

## まとめ

レギュラーキーやマルチシグの仕組みはEthereumのAccount Abstractionで実現可能と言われていますが、XRP Ledgerではすでにネイティブに実装されています。
資産を安全に管理するための機能がスマートコントラクトなしに利用できることはXRP Ledgerの強みの一つと言えるでしょう。


XRP Ledgerに興味がある方は開発者Discordチャンネルへ是非お越しください！
日本語チャンネルもありますので、英語が出来なくても大丈夫です！

https://discord.gg/aBH9MwcsbR