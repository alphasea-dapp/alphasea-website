---
title: "AlphaSeaとは？ v1 (old)"
description: "AlphaSea is a decentralized marketplace for market alphas."
date: 2022-01-18T00:00:00+00:00
draft: false
---

AlphaSeaは市場アルファのための分散型マーケットプレイスです。
Numeraiのようなことを分散型で実装するために作りました。

[AlphaSeaの仕組み](/how-it-works/)も見ると理解しやすいです。

## 全体像

{{< figure src="/svgs/overview.svg" title="AlphaSea overview" >}}

## メリット

### 運用資金をステークしなくていいので仮想通貨の価格変動リスクに晒されづらい

Predictorはガス代だけ、
Executorはガス代と予測購入費用だけウォレットに入れておけば良いので、
仮想通貨の価格変動リスクに晒されづらいです。
Executorは運用資金の大部分はCEX側に入れておくイメージです。

### トラストレス/Decentralized

- Polygon(matic)で実装
- コントラクトはimmutable(デプロイ後、誰も更新できない)
- コントラクトはpolygonscanでverify(ソースコードとの紐付け証明)済み
- ソースコードは全てオープンソース(CC0)

### 誰でも投資家側(Executor)になれる

誰でもExecutorになってメタモデルに基づいてトレードできます。
PredictorとExecutorを兼ねるとリスク分散になって良いと思います。

一部のPredictorの予測精度が悪くても、
半分くらいのPredictorの精度が良ければ、
メタモデルの成績は維持されると思います。
Executorをやれば、
botの汎化性能が出ない、
botの寿命、
または、botにバグがあって損失することがありますが、
そのようなリスクを減らせると思います。

### 仮想通貨が予測対象なので儲かりやすいはず

主要CEXでの売買代金が多く新しすぎない仮想通貨10銘柄の日足でトレードします。
銘柄一覧: BTC,ETH,XRP,LINK,ATOM,DOT,SOL,BNB,MATIC,ADA。
仮想通貨が対象なので、株よりも利益率が高くなりやすいはずです。
Executorが儲かれば、Predictorも儲かりやすくなります。

参加者全体の資金が増えてきて、
スケーラビリティーに問題が出てきたら、
1日3回にトーナメントを増やしたり、
株のトーナメントを追加したりすれば良いと思います。

### 真の貢献にインセンティブが与えられる

Executorは自分が儲けるために、
必要な予測のみを購入するので、
真の貢献にインセンティブが与えられます。

Predictorは成績さえ良ければ資金力が無くても稼げる可能性があります。
必要なのはガス代だけです。
反面、弱いモデルはガス代負けする可能性があります。

メタモデルのロジック詳細は[AlphaSeaの仕組み](/how-it-works/)参照

## Numeraiとの比較

AlphaSeaのメリット

- 誰でもメタモデルを使ってトレードできる
- 仮想通貨が対象
- MMCやTCの仕様に悩む必要が無い
- 真の貢献にインセンティブ
- NMRの価格変動リスクに晒されない

AlphaSeaのデメリット

- 資金が少ないとガス代負けするかもしれない
- example model(マイナーチェンジも含む)を投稿するだけではおそらく稼げない (真の貢献)
- データが少なくバリデーションが難しいという意味で難易度が高いかもしれない (株よりも銘柄少ない。歴史が浅い)
- 自分でagentなどを動かさないといけない

## コピートレードとの比較

AlphaSeaのメリット

- フォワード成績を元に、メタモデルで統合するので、手動よりも良いモデルを選べるはず
- 自動でモデルを選ぶので、モデルの移り変わりに対応できる (コピー対象が非アクティブになって止まるリスクが小さい)
- 執行前に情報を出すので、コピー対象の人と同じタイミングで執行できる (成績再現)
- 執行方法が統一化されるので、スケーラビリティーがある程度保証される
- 成績が悪かったらメタモデルから外れるだけなので、コピーさせる側の責任が小さい (レピュテーションリスク)

AlphaSeaのデメリット

- 資金が少ないとガス代負けするかもしれない。
- 執行方法が統一化されるので、トレードの自由度が小さい
- 予測してから執行までラグが大きい(1時間)ので予測が難しい (ブロックチェーン経由で予測売買するので時間が必要)

## botとの比較

AlphaSeaのメリット

- 自分の予測精度が低くてもトレード成績に影響を与えづらい
- 予測精度が良い場合は、自分のトレード収益に加えて、予測販売収益を得られる

AlphaSeaのデメリット

- トレードの自由度が小さい
- 予測してから執行までラグが大きい(1時間)ので予測が難しい (ブロックチェーン経由で予測売買するので時間が必要)

全員の戦略をお互いにロジックを開示すること無しにアンサンブルして、
全員で恩恵を受けられたらいいなと思っていました。
PredictorとExecutorを両方やる想定です。
ロジックを開示しないので、
マーケットインパクトで利益が毀損されるなどの理由で、
辞めたくなったらいつでも辞められます。
