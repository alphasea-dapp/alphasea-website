---
title: "AlphaSeaの仕組み"
description: "AlphaSea is a decentralized marketplace for market alphas."
date: 2022-01-17T00:00:00+00:00
draft: false
---

## 全体像

{{< figure src="/svgs/overview.svg" title="AlphaSea overview" >}}

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
- [thegraph.com alphasea-polygon](https://thegraph.com/hosted-service/subgraph/richmanbtc/alphasea-polygon)

### alphasea.io

#### ui

リーダーボードです。

- [alphasea-dapp/alphasea-ui](https://github.com/alphasea-dapp/alphasea-ui)
- [Leaderboard](https://app.alphasea.io/)

#### website

このウェブサイトです。

[alphasea-dapp/alphasea-website](https://github.com/alphasea-dapp/alphasea-website)

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

## トーナメント

2時間ずらしで1日に12ラウンド。
各ラウンドのポジション持続期間は24H。
常に12ラウンド分のポジションが被るので、
それらを平均化したものをメタモデルのポジションとする。
詳細は[AlphaSeaの仕組み](/how-it-works)参照。

| 予測提出締切 | TWAP執行エントリー | TWAP執行エグジット |
|:-:|:-:|:-:|
|00:08|00:30-02:30|24:30-26:30|
|02:08|02:30-04:30|26:30-28:30|
|04:08|04:30-06:30|28:30-30:30|
|06:08|06:30-08:30|30:30-32:30|
|08:08|08:30-10:30|32:30-34:30|
|10:08|10:30-12:30|34:30-36:30|
|12:08|12:30-14:30|36:30-38:30|
|14:08|14:30-16:30|38:30-40:30|
|16:08|16:30-18:30|40:30-42:30|
|18:08|18:30-20:30|42:30-44:30|
|20:08|20:30-22:30|44:30-46:30|
|22:08|22:30-24:30|46:30-48:30|

時刻はUTC。

Executorで資産が少ない場合はガス代を節約するために、
00:30執行のもののみ購入なども考えられる。
トーナメントが多いのは執行を24Hに分散させてスケールさせるためなので、
購入するトーナメントが少なくても成績はあまり変わらないはず。
ただし、アンサンブル効果で多少は成績改善するので、
資産が多い場合は全てを購入するのが得だと思う。

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

## 市場規模試算

条件

- 10銘柄 (BTC,ETH,XRP,LINK,ATOM,DOT,SOL,BNB,MATIC,ADA)
- 売買代金: $200M/銘柄/day
- 売買代金に対する利益率: 0.002
- 全体に対する自身の売買代金の割合: 0.1

結果

月次13億円

$200M * 30日 * 10銘柄 * 0.1 * 0.002 = $12M/month = 13億円/month

### 参考データ

#### 売買代金

1銘柄あたり$200M/dayとおいた。
一つの取引所に集中させる必要は無いので、保守的な見積もり。

```text
24H volume (2022/01/21)
BTC/USDT $12,208,611,310
ETH/USDT $4,514,856,665
ADA/USDT $1,582,590,923
ATOM/USDT $684,507,517
BNB/USDT $557,588,011
MATIC/USDT $485,564,570
SOL/USDT $483,956,038
LINK/USDT $424,261,421
DOT/USDT $366,274,080
XRP/USDT $258,954,988
```
source: [coinmarketcap binance perp](https://coinmarketcap.com/exchanges/binance/?type=perpetual)

#### 売買代金に対する利益率

直接的に計算していないが、
取引コスト0.001でバックテストして、
ret with costがcostの4倍くらいなので、
売買代金に対する利益率は0.004くらい。
保守的に0.002とした。

[alphasea-example-model/notebooks/example_model_rank.ipynb](https://github.com/alphasea-dapp/alphasea-example-model/blob/master/notebooks/example_model_rank.ipynb)

#### 全体に対する自身の売買代金の割合

過去、取引所の売買代金の10%くらいをボットでトレード(maker)していたことがある。
マーケットインパクトなのか偶然なのか、
少し儲かりづらかった気がするが、
稼働していた実績があるので、10%とおいた。

## ガス代試算

条件

- ブロックチェーン: Polygon
- Predictor: 100人
- Predictor予測数: 1/人/tournament
- Predictor gas: 100000/prediction
- Executor: 100人
- Executor購入数: 5/人/tournament
- Executor gas: 60000/purchase
- トーナメント数: 12/日
- gas price: 400gwei
- MATIC price: 250円

結果

1人

- Predictor gas fee: 3600円/月
- Executor gas fee: 10800円/月

全体

- Predictor total gas fee: 36万円/月
- Executor total gas fee: 108万円/月
- AlphaSea total gas fee: 144万円/月

00:30執行ラウンドのみ参加なら1/12になる。

最初、ETHで作りましたが、試算したところガス代負けしそうなので、
EVM互換の中で最も馴染みがありそうなPolygonに変えました。

## TWAP執行時間と執行ラグ

簡単な予測モデルで色々試して、
TWAP執行時間は2H、ラグは0.5Hにしました。

### TWAP執行時間

- 長いとガス代を節約できる (24Hを網羅するためのトーナメント数が少ない)
- 短いと予測しやすい

執行時間が4Hだと2Hより少し予測精度が落ちる。
執行時間が1Hだとガス代が2倍になる割に予測精度が2Hとあまり変わらない。

### 執行ラグ

- 長いと取引や計算時間に余裕がある (パフォーマンスチューニングしなくて良い。トラブルの可能性が減る)
- 短いと予測しやすい

ラグが15minと30minだと予測精度があまり変わらない。
1Hだと30minより少し精度が落ちる。

## トーナメントは拡張可能

トーナメントの質はプラットフォームの質とは独立です。

プラットフォーム(AlphaSeaやnumerai)の質

- 予測精度 (メタモデル。インセンティブ設計など)
- 参加コスト (ガス代、ファンドfee、NMRリスクなど)
- 思想 (decentralized。オープンソース)

トーナメントの質

- 利益率
- シャープレシオ
- スケーラビリティー

もしAlphaSeaが機能することを確認できたら、
トーナメントはニーズに応じて変えたり追加していけば良いと思います。
例えば、仮想通貨なら以下のような設定が考えられます。
時間軸を伸ばせばもっとスケールすると思います。
株, dex, defiなどもありえます。

### BTCのみの場合

条件

- 1銘柄
- 売買代金: $12B/銘柄/day
- 売買代金に対する利益率: 0.002
- 全体に対する自身の売買代金の割合: 0.1

結果

$12B * 30日 * 1銘柄 * 0.1 * 0.002 = $72M/month = 82億円/month

### BTC, ETHの場合

条件

- 2銘柄
- 売買代金: $4B/銘柄/day
- 売買代金に対する利益率: 0.002
- 全体に対する自身の売買代金の割合: 0.1

結果

$4B * 30日 * 1銘柄 * 0.1 * 0.002 = $24M/month = 27億円/month
