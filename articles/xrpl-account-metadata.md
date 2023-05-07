<!--
title:   XRPLアカウントのメタデータ
tags:    Blockchain,XRPLedger,web3,xrp
private: false
-->

## はじめに

XRPレジャーは、分散型台帳技術を採用したブロックチェーンの1つです。2012年にXRP Ledgerは3人の開発者によって開発され、現在もXRPL財団やRipple社、XRPL Labsなどを始めとしたXRPLコミュニティによって開発が続けられています。

XRPLのアカウントは単に資産の残高データを持つだけでなく、他のデータも保有することができます。

## アカウント

XRPレジャーのアカウントにはいくつかのメタデータを設定することができます。これらを設定することで他者がアカウントの所有者の情報を確認することが可能になります。
このようなデータはアカウント情報を保持する`AccountRoot`オブジェクトに格納されます。

```json :AccountRoot
{
  "LedgerEntryType": "AccountRoot",
  "Account": "rfe5EiXPyruyrM47jRKkZX7Gx81DrEDuy6",
  "Balance": "999999904",
  "Domain": "6578616D706C652D646F6D61696E2E636F6D",
  "EmailHash": "CF1D201E1FBEA4E38CE832149C0BAB4A",
  "Flags": 0,
  "MessageKey": "020000000000000000000000004CCAE8EBCB878C8DB19A910A5EEBCE32E8693211",
  "OwnerCount": 0,
  "PreviousTxnID": "819CBB4911DE6F354947BA9D9745A3B27B50B91E93F0D467CA7B42C3D63D7F8F",
  "PreviousTxnLgrSeq": 37608192,
  "Sequence": 37607227,
  "index": "D41F5D7BADDBBBBB807804FC0AC6A97EA52775639C5D530F7539B1D790DBBF00",
}
```

### メタデータ

#### Domain

`Domain`フィールドには任意のドメインを表す文字列を設定することができます。
いくつかのサービスではこの情報を利用し、アカウントに紐付くドメインとして表示しています。

このフィールドのドメインを設定するだけではドメインの所有者がこのアカウントを保持することの証明にはなリません。そのドメインのウェブサイトにてXRPレジャーに関する情報を公開することで双方向リンクを確立することが推奨されています。
慣例上、その情報は`{ドメイン名}/.well-known/xrp-ledger.toml`で公開します。

#### EmailHash

`EmailHash`フィールドにはアバター画像の生成のために使用するメールアドレスのハッシュ値を指定します。一般的には[Gravatar](https://ja.gravatar.com/site/implement/hash/)を利用しアバターの生成が行われます。

次のURLでGravatar画像の取得を行うことができます。。
https://secure.gravatar.com/avatar/{EmailHashを小文字で指定}

#### MessageKey

`MessageKey`フィールドには任意の公開鍵を設定することができます。鍵の種類を表す先頭1バイト+任意の32バイトの合計33バイトを設定します。用途などは自由です。

過去にはFlare NetworkがXRPLアドレスの残高をベースにトークン配布を行う際、XRPLアドレスに紐付けるトークン配布先のイーサリアムアドレスを設定するために利用されました。

#### WalletLocator

`WalletLocator`には任意の256ビット値を指定することができます。

#### WalletSize

`WalletSize`には任意の32bit整数値を指定することができます。

https://xrpl.org/ja/accountset.html#accountset-%E3%83%95%E3%82%A3%E3%83%BC%E3%83%AB%E3%83%89

## メタデータを利用するサービス

いくつかのサービスではDomainやEmailHashの情報を参照し、アカウントの情報として表示しています。

### XRPScan

![xrpscan](https://user-images.githubusercontent.com/69445828/236678843-ac2374fa-47b5-4e4b-97f9-178959330004.png)

https://xrpscan.com/account/rQQQrUdN1cLdNmxH4dHfKgmX5P4kf3ZrM

### bithomp

![bithomp](https://user-images.githubusercontent.com/69445828/236678863-f0f355cd-fc50-4d47-b1de-11e653b8f026.png)

https://bithomp.com/explorer/rQQQrUdN1cLdNmxH4dHfKgmX5P4kf3ZrM

## メタデータの設定

これらのメタデータは`AccountSet`トランザクションを使用し、設定・更新・削除を行うことができます。

ここではXRPレジャーのJavascriptライブラリである`xrpl.js`を使った例を紹介します。

https://www.npmjs.com/package/xrpl

```js
import { Client, Wallet, convertStringToHex } from "xrpl";
const crypto = require('crypto')

const md5hex = (str: string) => {
  const md5 = crypto.createHash('md5')
  return md5.update(str, 'binary').digest('hex')
}

const wallet = Wallet.fromSecret("sEdSS1xTNskRyjzBjBbHMbJW6SsYv7J");
// rfe5EiXPyruyrM47jRKkZX7Gx81DrEDuy6

const client = new Client("wss://testnet.xrpl-labs.com");

const main = async () => {
  await client.connect();
  const response = await client.submitAndWait(
    {
      TransactionType: "AccountSet",
      Account: "rfe5EiXPyruyrM47jRKkZX7Gx81DrEDuy6",
      Domain: convertStringToHex("example-domain.com"),
      EmailHash: md5hex('alice@example-email.com'.trim()).toUpperCase(),
      MessageKey: '020000000000000000000000004CCAE8EBCB878C8DB19A910A5EEBCE32E8693211',
    },
    { wallet }
  );
  console.log(response.result.hash)
};

main()
```

- Domain: Hex値で設定
- EmalHash: メールアドレスをmd5でハッシュ化し設定

## まとめ

XRPレジャーのアカウントにはいくつかのメタデータが設定可能であることを説明しました。メタデータは非常に簡単にかつ低手数料で設定/変更することができます。

メタデータはAccountRootオブジェクトの一部であり、XRPLのバリデータの合意があればフィールドの追加も可能です(コアプログラムであるrippledの修正が必要です)。

XRP Ledgerに興味がある方は開発者Discordチャンネルへ是非お越しください！
日本語チャンネルもありますので、英語が出来なくても大丈夫です！

https://discord.gg/aBH9MwcsbR
