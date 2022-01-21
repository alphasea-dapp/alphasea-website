---
title: "AlphaSeaトーナメント"
description: "AlphaSea is a decentralized marketplace for market alphas."
date: 2022-01-21T00:00:00+00:00
draft: false
---

## トーナメント

2時間ずらしで12個のトーナメント。
各トーナメントは毎日行う。
crypto_daily_0030の場合、
0:30-2:30UTCの平均価格から次の日の0:30-2:30UTCの平均価格までが1日で得られるリターン。
詳細は[AlphaSeaの仕組み](/how-it-works)参照。

| tournament_id | 予測提出締切 | TWAP執行時間 |
|:-:|:-:|:-:|
|crypto_daily_0030|00:08|00:30-02:30|
|crypto_daily_0230|02:08|02:30-04:30|
|crypto_daily_0430|04:08|04:30-06:30|
|crypto_daily_0630|06:08|06:30-08:30|
|crypto_daily_0830|08:08|08:30-10:30|
|crypto_daily_1030|10:08|10:30-12:30|
|crypto_daily_1230|12:08|12:30-14:30|
|crypto_daily_1430|14:08|14:30-16:30|
|crypto_daily_1630|16:08|16:30-18:30|
|crypto_daily_1830|18:08|18:30-20:30|
|crypto_daily_2030|20:08|20:30-22:30|
|crypto_daily_2230|22:08|22:30-00:30|

時刻はUTC。

Executorで資産が少ない場合はガス代を節約するために、
crypto_daily_0030のみ購入なども考えられる。
トーナメントが多いのは執行を24Hに分散させてスケールさせるためなので、
購入するトーナメントが少なくても成績はあまり変わらないはず。
ただし、アンサンブル効果で多少は成績改善するので、
資産が多い場合は全てを購入するのが得だと思う。

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

crypto_daily_0030のみ参加なら1/12になる。

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
