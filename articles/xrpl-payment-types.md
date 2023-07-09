<!--
title:   XRP Ledgerの強力なPayment機能
tags:    Blockchain,Web3,XRPLedger,xrp
id:      79c896f053768842fa3b
private: false
-->
XRP Ledgerは強力な支払い機能を持つパブリックなブロックチェーンです。

支払い機能には次の種類のトランザクションが存在します。

#### Payment

XRPおよび発行トークンでの直接支払い、クロスカレンシー支払いなどを行います。

#### [Checks](https://xrpl.org/ja/checks.html)

XRPおよび発行トークンで非同期で支払い、転送を行います。支払人は任意の通貨(XRPや発行トークン)でCheckを発行でき、受取人は支払人が指定した通貨や、DEX流動性を利用して他の通貨に変換して受け取ることもできます。

#### [Escrow](https://xrpl.org/ja/escrow.html)

XRPを利用し、条件付き支払いを行います。終了時間ベースのエスクロー、暗号条件ベースでのエスクロー、この2つを組み合わせたエスクローの3つのケースが利用可能です。
発行トークンでEscrowを利用可能にする機能も開発中です。([XLS-34d](https://github.com/XRPLF/XRPL-Standards/discussions/88))

#### [Payment Channel](https://xrpl.org/ja/payment-channels.html)

少額の単位に分割可能な「非同期」のXRPペイメントを送信し、後日決済を行います。送金は個別に検証されるため、XRP Ledgerのトランザクション処理能力を超える速度での支払いが可能です。Payment Channelの終了時にXRP Ledgerにて一括清算を行います。
発行トークンでPayment Channelを利用可能にする機能も開発中です。([XLS-34d](https://github.com/XRPLF/XRPL-Standards/discussions/88))

## Payment トランザクション

今回はその支払い機能の中での根幹となるPaymentトランザクションに注目して説明します。

[Payment - XRPL.org (日本語)](https://xrpl.org/ja/payment.html)

PaymentトランザクションではXRP LedgerのネイティブトークンであるXRPだけではなく、ユーザが発行できる独自のトークン(発行トークン/IOUなどと呼ばれます)も利用することができます。

## Paymentトランザクションの例

```json
{
  "TransactionType" : "Payment",
  "Account" : "rs7FV1WHDutSL8o6iNy1e1quf7Daf8CNrk",
  "Destination" : "ra5nK24KXen9AHvsdFTKHSANinZseWnPcX",
  "Amount" : {
     "currency" : "USD",
     "value" : "1",
     "issuer" : "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn"
  },
  "Fee": "12",
  "Flags": 2147483648,
  "Sequence": 2,
}
```

## 主なフィールド

| フィールド | 説明 |
| --- | --- |
| Amount | 送金先への送金通貨・送金額 |
| Destination | 送金先 |
| DestinationTag | 送金先を識別するためのタグ |
| Paths | 支払いが送金元から受取人に届くまでにたどる経路 |
| SendMax | 送金するために利用する送金元の最大の通貨・額 |
| DeliverMin | 送金先への最小の送金金額 |

## 主な機能

- XRP直接決済
  - 非常に一般的な決済方法
  - XRP-XRP
- クロスカレンシー決済
  - XRP-IOU(発行トークン)
  - IOU(発行トークン)-IOU(発行トークン)
    - DEXを利用した通貨の自動変換も可能
- 通貨変換
  - Account = Destination
  - いわゆるSwap

## 特殊なフィールド

### Amount

- Destinationが受け取る通貨・額を表します。

### SendMax

- Accountが送金する通貨・額を表します。
- 省略されている場合はAmountと同じ通貨・額となります。

### DeliverMin

- Partial Paymentが有効な場合にのみ使用可能です。
- Destinationが受け取る最小の金額を指定します。
- Destinationが受け取る金額は指定値に満たない場合はトランザクションはエラーとなります。

### Partial Payment (Flagsフィールドで指定)

- Destinationに送金する金額がAmountに指定した金額に満たない場合でも送金可能な額まで送金します。

## 種類ごとのトランザクション例

### XRP直接決済

- *XRP-XRP*

  AmountにXRPをdropの単位(1XRP=1000000drops)で指定します。

```json
  {
    "TransactionType" : "Payment",
    "Account" : "rs7FV1WHDutSL8o6iNy1e1quf7Daf8CNrk",
    "Destination" : "ra5nK24KXen9AHvsdFTKHSANinZseWnPcX",
    "Amount" : "1000000", // 1XRP
    "Fee": "12",
    "Flags": 2147483648,
    "Sequence": 2,
  }
```

### クロスカレンシー決済

- **XRP-IOU(発行トークン)**

  1USDを送信するために最大10XRPを送金します。
  送金の過程でDEXを利用し、XRPをUSDに変換します。

  1USDを送信するために2XRPしか使用しなかった場合、残りの8XRPは送信元アカウントに返されます。

```json
    {
      "TransactionType" : "Payment",
      "Account" : "rs7FV1WHDutSL8o6iNy1e1quf7Daf8CNrk",
      "Destination" : "ra5nK24KXen9AHvsdFTKHSANinZseWnPcX",
      "Amount" : {
        "currency" : "USD",
        "value" : "1",
        "issuer" : "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn"
      },
      "SendMax": "10000000", // 10XRP
      "Fee": "12",
      "Flags": 2147483648,
      "Sequence": 2,
    }
```

- **IOU（発行トークン）-IOU（発行トークン）**

  1USDを利用するために最大1EURを送金します。
  送金の過程でDEXを利用し、USDをEURに変換します。
  USD/EURの流動性が小さい場合、USD/XRPやEUR/XRPの流動性も使用します。

```json
    {
      "TransactionType" : "Payment",
      "Account" : "rs7FV1WHDutSL8o6iNy1e1quf7Daf8CNrk",
      "Destination" : "ra5nK24KXen9AHvsdFTKHSANinZseWnPcX",
      "Amount" : {
        "currency" : "USD",
        "value" : "1",
        "issuer" : "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn"
      },
      "SendMax": {
        "currency" : "EUR",
        "value" : "1",
        "issuer" : "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn"
      },
      "Fee": "12",
      "Flags": 2147483648,
      "Sequence": 2,
    }
```

### 通貨変換

Destinationを自身のアカウントとすることで、自身が保有する通貨を別の通貨へ変換することができます。
このトランザクションも他と同様にDEXを利用して変換を行います。

```json
  {
    "TransactionType" : "Payment",
    "Account" : "rs7FV1WHDutSL8o6iNy1e1quf7Daf8CNrk",
    "Destination" : "rs7FV1WHDutSL8o6iNy1e1quf7Daf8CNrk",
    "Amount" : {
      "currency" : "USD",
      "value" : "1",
      "issuer" : "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn"
    },
    "SendMax": {
      "currency" : "EUR",
      "value" : "1",
      "issuer" : "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn"
    },
    "Fee": "12",
    "Flags": 2147483648,
    "Sequence": 2,
  }
```

### PartialPayment

FlagsにtfPartialPaymentを設定することでPartialPaymentが有効化されます。
10USDを利用するために最大1EURを送金する場合、1EURをUSDに変換した結果10USDに満たない場合でも、変換した分のUSD分を全て送信します。

```json
  {
    "TransactionType" : "Payment",
    "Account" : "rs7FV1WHDutSL8o6iNy1e1quf7Daf8CNrk",
    "Destination" : "ra5nK24KXen9AHvsdFTKHSANinZseWnPcX",
    "Amount" : {
      "currency" : "USD",
      "value" : "10",
      "issuer" : "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn"
    },
    "SendMax": {
      "currency" : "EUR",
      "value" : "1",
      "issuer" : "rwqHQ87A8b5WjobXfVzvXiKAGYePoRfapa"
    },
    "Fee": "12",
    "Flags": 2147614720, // tfFullyCanonicalSig(2147483648) + tfPartialPayment(131072)
    "Sequence": 2,
  }
```

## パス

Pathsフィールドを設定することで、通貨の変換時に利用するオーダーブック等を指定できます。これによりクロスカレンシー決済や通貨変換時により低いコストで通貨の変換を実現することができます。
パスには複数の経路を含めることができ、例えばUSDをEURへ変換する場合に(USD/EUR)と(USD/XRP,EUR/XRP)のオーダーブックを利用することができます。

## まとめ

XRP LedgerのPaymentトランザクションにはXRPの送金だけでなく通貨変換など様々な利用方法があります。
Paymentトランザクション以外にもNFTやDEXでのトレードなどの様々な種類のスマートトランザクタ(プロトコルで提供されているネイティブな機能)が存在します。

ユーザが独自のロジックをアカウントに紐付けられるスマートコントラクト機能(Hooks)やAMM機能などもコミュニティによって開発中です。

興味を持たれた方はXRP Ledger開発者のDiscordチャンネルへ是非お越しください！
日本語チャンネルもありますので、英語が出来なくても大丈夫です！

https://discord.gg/aBH9MwcsbR