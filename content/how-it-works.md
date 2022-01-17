---
title: "AlphaSeaの仕組み"
description: "AlphaSea is a decentralized marketplace for market alphas."
date: 2022-01-17T00:00:00+00:00
draft: false
---

## 登場人物

### Predictor

予測を売る人です。
Numeraiだとシグナルを投稿する人に相当します。

### Executor

予測を買って、メタモデルで統合し、トレードする人です。
NumeraiだとNumerai fundに相当します。

## 取引方法

ETHを介して予測データを売買します。
カッコ内の英語はソースコードで使われている名前です。

### タイムライン

取引の流れは以下のようになっています。時刻はUTCです。
これを毎日繰り返します。

- 0:00-0:15 Predictorが予測投稿 (Create Prediction)
- 0:15-0:30 Executorが予測購入 (Purchase)
- 0:30-0:45 Predictorが予測送信 (Ship)
- 1:00-2:00 ExecutorがTWAP執行 (CEXなど。AlphaSeaの管轄外)
- 2:00-2:15 Predictorが予測公開 (Publish)

### 予測投稿 (Create Prediction)

Predictorが予測を投稿します。
具体的には、共通鍵(contentKey)で暗号化した予測(encryptedContent)をETH上に書き込みます。
この時点では予測は暗号化されているので、 投稿者以外は見れません。
暗号化はNaClのSecretBoxを使っています。

モデルIDと秘密のデータ(contentKeyGenerator)から計算したハッシュを共通鍵(contentKey)として使います。
他人の予測をコピーする攻撃を防ぐためのものです。
詳細は後述の攻撃で説明します。

### 予測を購入 (Purchase)

Executorが予測を購入します。
購入時にShip用の公開鍵(publicKey)をETH上に書き込みます。

補足

- この公開鍵はウォレットの公開鍵とは異なります。

### 購入者へ予測データを送る (Ship)

Predictorが購入者へ予測データを送ります。
具体的には、Ship用の公開鍵で暗号化した共通鍵(encryptedContentKey)をETH上に書き込みます。
購入者のみがencryptedContentKeyを復号化しcontentKeyを得られます。
contentKeyでencryptedContentを復号化すれば予測を見れます。
暗号化はNaClのBoxを使っています。

### 予測データの公開 (Publish)

Predictorが執行時間が終わったあとで予測を公開します。
具体的には共通鍵(contentKey)をETH上に書き込みます。
誰でも予測を見れるようになります。
次回以降、Executorが予測を購入するときの検討材料になります。

## 攻撃と対策

### 他人の予測をコピー

他人の予測をコピーする攻撃です。
具体的には以下の手順で行います。

1. 投稿された予測と同じデータ(暗号化されている)を投稿
2. 公開された予測と同じデータをPublish

対策

モデルIDと秘密のデータ(contentKeyGenerator)から計算したハッシュを共通鍵として使う。

モデルIDをハッシュ計算に含めるので、
2で同じデータをPublishしても、
計算される共通鍵が異なる。
正常に復号化できるかどうかで、
オリジナルの予測かどうかを判定できる。

### 適当な予測を投稿しまくる

適当な予測を投稿しまくり、
偶然フォワード成績の良い予測を産み出し、
購入されることを期待する攻撃です。

対策

ガス代が対策になる。

予測の数に比例してガス代がかかるので、
適当な予測を投稿しすぎると、ガス代負けして損失するので、
攻撃が抑制されます。

## アーキテクチャー

{{< figure src="/svgs/architecture.svg" title="AlphaSea architecture" >}}

Decentralizedに実装されています。
色の付いたexample-modelが特に重要で、
Predictorが各自改善する部分です。

### Your Server

Predictor, Executorをやる人が管理するサーバーです。
AlphaSeaに参加したい人はこれを用意すれば参加できます。
AWSやGCPで動かすと良いと思います。

#### agent

alphasea-agentは、AlphaSeaスマートコントラクトを、
シンプルなインターフェースで扱えるようにするための、
HTTPサーバーです。
Predictor, Executorをやる人はこれを動かす必要があります。
予測購入の判断ロジックやメタモデルはagentに組み込まれています。

[alphasea-dapp/alphasea-agent](https://github.com/alphasea-dapp/alphasea-agent)

#### example-model

alphasea-example-modelは、alphasea-agent に対して毎日予測を投稿するプログラムです。
Numeraiのexample modelに相当します。
Predictorをやる人は、これをベースに改善すると良いと思います。

[alphasea-dapp/alphasea-example-model](https://github.com/alphasea-dapp/alphasea-example-model)

#### trade-bot

alphasea-trade-botは、
alphasea-agentから毎日メタモデル予測結果を取得してCEXのPERPポジションをリバランスするプログラムです。
Executorをやる人は、これを動かせば良いです。

[alphasea-dapp/alphasea-trade-bot](https://github.com/alphasea-dapp/alphasea-trade-bot)

### AlphaSea Contract

AlphaSeaのスマートコントラクトです。
予測データの投稿、購入、送信、公開を処理します。

[alphasea-dapp/alphasea](https://github.com/alphasea-dapp/alphasea)

### thegraph.com

thegraphはスマートコントラクトがemitしたEventを溜めて、
GraphQLでクエリーできるようにするものです。
thegraph.comのhosted serviceを使っています。
リーダーボードから使っています。

- [alphasea-dapp/alphasea/subgraph](https://github.com/alphasea-dapp/alphasea/tree/master/subgraph)
- [thegraph.com alphasea-ropsten](https://thegraph.com/hosted-service/subgraph/richmanbtc/alphasea-ropsten)

### alphasea.io

#### ui

リーダーボードです。

- [alphasea-dapp/alphasea-ui](https://github.com/alphasea-dapp/alphasea-ui)
- [Leaderboard (testnet)](https://alphasea-app-ropsten.netlify.app/)

#### website

このウェブサイトです。

[alphasea-dapp/alphasea-website](https://github.com/alphasea-dapp/alphasea-website)

## トーナメントルール

毎日、以下のようなデータを提出します。

```text
symbol,position
BTC,0.123
ETH,-0.04
XRP,0.03
LINK,0.2
ATOM,-0.01
DOT,0.04
SOL,0.11
BNB,-0.21
MATIC,0.04
ADA,-0.05
```

positionはリバランス後のポジションを表します。
プラスはロング、マイナスはショートです。
ポジションの絶対値の合計は1以下です。
リバランスは1:00-2:00UTCの1時間でTWAP執行によって行うことを想定しています。

[crypto_daily.md](https://github.com/alphasea-dapp/alphasea/blob/master/tournaments/crypto_daily.md)

## 予測データライセンス

第三者が自由に利用できるようにCC0-1.0のみ。
予測投稿時に指定。

## データ保存場所

予測データはETHにオンチェーンで保存されます。
一時的なものはagentプロセスのメモリ上に保存されます。

|データ|保存場所|ライフサイクル|
|:-:|:-:|:-:|
|予測データ|ETH|永久|
|公開前の予測データ|agentプロセスのメモリ|agentプロセス終了まで|
|予測暗号化の鍵|agentプロセスのメモリ|agentプロセス終了まで|
|購入データ|ETH|永久|

ETHに何が保存されているかは、以下のデバッグページを見るとイメージをつかめると思います。

[デバッグ (testnet)](https://alphasea-app-ropsten.netlify.app/debug)
