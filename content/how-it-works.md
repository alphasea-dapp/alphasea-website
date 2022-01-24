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

polygon(matic)上のスマートコントラクトを介して予測データを売買します。
カッコ内の英語はソースコードで使われている名前です。

### タイムライン

取引の流れは以下のようになっています。時刻はUTCです。
これを毎日繰り返します。

- 00:00-00:08 Predictorが予測投稿 (Create Prediction)
- 00:08-00:16 Executorが予測購入 (Purchase)
- 00:16-00:24 Predictorが予測送信 (Ship)
- 00:30-02:30 ExecutorがTWAP執行 (CEXなど。AlphaSeaの管轄外)
- 26:30-26:45 Predictorが予測公開 (Publish)

これは、00:30執行ラウンドの場合です。
[AlphaSeaトーナメント](/tournament)参照。

### 予測投稿 (Create Prediction)

Predictorが予測を投稿します。
具体的には、共通鍵(contentKey)で暗号化した予測(encryptedContent)をブロックチェーン上に書き込みます。
この時点では予測は暗号化されているので、 投稿者以外は見れません。
暗号化はNaClのSecretBoxを使っています。

モデルIDと秘密のデータ(contentKeyGenerator)から計算したハッシュを共通鍵(contentKey)として使います。
他人の予測をコピーする攻撃を防ぐためのものです。
詳細は後述の攻撃で説明します。

### 予測を購入 (Purchase)

Executorが予測を購入します。
購入時にShip用の公開鍵(publicKey)をブロックチェーン上に書き込みます。

補足

- この公開鍵はウォレットの公開鍵とは異なります。

### 購入者へ予測データを送る (Ship)

Predictorが購入者へ予測データを送ります。
具体的には、Ship用の公開鍵で暗号化した共通鍵(encryptedContentKey)をブロックチェーン上に書き込みます。
購入者のみがencryptedContentKeyを復号化しcontentKeyを得られます。
contentKeyでencryptedContentを復号化すれば予測を見れます。
暗号化はNaClのBoxを使っています。

### 予測データの公開 (Publish)

しばらく経ったあとにPredictorが予測を公開します。
具体的には共通鍵(contentKey)をブロックチェーン上に書き込みます。
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

### 予測を購入せずに公開後タダ乗りする

予測が公開されたあとに予測に従ってトレードする攻撃です。
予測を購入する場合よりもエントリーが遅れますが、
リターンを得られる可能性があります。

対策

予測の公開を遅らせる

最初のアイデアでは2:00-2:15UTCに予測公開でしたが、
1日遅らせて、26:00-26:15UTCにしました。

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
- [thegraph.com alphasea-mumbai](https://thegraph.com/hosted-service/subgraph/richmanbtc/alphasea-mumbai)

### alphasea.io

#### ui

リーダーボードです。

- [alphasea-dapp/alphasea-ui](https://github.com/alphasea-dapp/alphasea-ui)
- [Leaderboard (testnet)](https://alphasea-app-mumbai.netlify.app/)

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
リバランスは0:30-2:30UTCの1時間でTWAP執行によって行うことを想定しています。
なので、0:30-2:30UTCの平均価格から次の日の0:30-2:30UTCの平均価格までが1日で得られるリターンです。

[crypto_daily.md](https://github.com/alphasea-dapp/alphasea/blob/master/tournaments/crypto_daily.md)

## 予測データライセンス

第三者が自由に利用できるようにCC0-1.0のみ。
予測投稿時に指定。

## データ保存場所

予測データはブロックチェーンにオンチェーンで保存されます。
一時的なデータはredis(agentとプロセスと一緒にdocker-composeで起動。disk永続化有り)に保存されます。

|データ|保存場所|ライフサイクル|
|:-:|:-:|:-:|
|予測データ|ブロックチェーン|永久|
|購入データ|ブロックチェーン|永久|
|予測暗号化の鍵|redis (disk永続化)|データ削除するまで|
|公開前の予測データ|redis (disk永続化)|48H|
|モデル選択結果|redis (disk永続化)|48H|
|イベントキャッシュ|redis (disk永続化)|データ削除するまで|

ブロックチェーンに何が保存されているかは、以下のデバッグページを見るとイメージをつかめると思います。

[デバッグ](https://app.alphasea.io/debug)

## メタモデル

メタモデルはalphasea-agentで実装されています。
現状の実装は以下のようになっています。

- 過去60日間の成績で評価 (成績が確定していない直近1日は除く)
- アンサンブルしたときのシャープレシオが最大となるように複数モデルを選択
- 購入したモデルを等ウェイトでアンサンブル
- 購入費用を考慮 (購入することで得られるリターンより、購入費用が多い場合は購入しない意図。ガス代は未実装)
- 取引コストを考慮

実装箇所

- [alphasea-agent/src/executor/executor.py](https://github.com/alphasea-dapp/alphasea-agent/blob/master/src/executor/executor.py)
- [alphasea-agent/src/model_selection/equal_weight_model_selector.py](https://github.com/alphasea-dapp/alphasea-agent/blob/master/src/model_selection/equal_weight_model_selector.py)

## 自動価格調整

予測の販売価格は以下のアルゴリズムで自動調整されます。
数値はalphasea-agentの設定で変えられます。

- 前回購入が0個: 価格を20%減らす
- 前回購入が1個以上: 価格を20%増やす
