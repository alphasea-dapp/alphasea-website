---
title: "AlphaSea Tutorial"
description: "AlphaSea is a decentralized marketplace for market alphas."
date: 2022-01-24T00:00:00+00:00
draft: false
---

AlphaSea (Polygon mainnet)用のチュートリアルです。

- [AlphaSeaについて](/introduction)
- [AlphaSeaの仕組み](/how-it-works)

## このチュートリアルでやること

1. alphasea-agentを動かす (2, 3を動かすために必要)
2. alphasea-example-modelを動かす (Predictorをやるために必要)
3. alphasea-trade-botを動かす (Executorをやるために必要)

注意

- このチュートリアルはpolygonのmainnetで動かすので、実際にガス代などがかかります
- alphasea-trade-botを動かすとCEXトレードで損失する可能性があります
- 自己責任

## Google Compute Engineインスタンスを作る

以下の構成のインスタンスを2つ作ります。
インスタンスA, Bと呼びます。

- マシンタイプ: e2-medium
- OS: Container optimized OS 93 LTS (docker version 20.10.0以上 [補足](https://qiita.com/skobaken/items/03a8b9d0e443745862ac))
- ディスク: 40GB

インスタンスAはインスタンスBから参照されるので、
[静的内部 IP アドレスの予約](https://cloud.google.com/compute/docs/ip-addresses/reserve-static-internal-ip-address)
をすると良いかもしれません。

## docker-composeをインストール

インスタンスA, Bで以下を実行し、docker-composeをインストール

```bash
echo alias docker-compose="'"'docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v "$PWD:$PWD" \
    -w="$PWD" \
    docker/compose:1.24.0'"'" >> ~/.bashrc
source ~/.bashrc
```

source: [Running Docker Compose with Docker](https://cloud.google.com/community/tutorials/docker-compose-on-container-optimized-os)

## インスタンスAでalphasea-agentを動かす

[alphasea-agent](https://github.com/alphasea-dapp/alphasea-agent) を動かします。

```bash
cd ~/
git clone https://github.com/alphasea-dapp/alphasea-agent.git
```

以下に従い秘密鍵を用意し、ウォレットにMATICを入れる。

[alphasea-agent#秘密鍵](https://github.com/alphasea-dapp/alphasea-agent#%E7%A7%98%E5%AF%86%E9%8D%B5%E3%82%92%E7%94%A8%E6%84%8F)

.envファイル(設定)を作成します。
```text
ALPHASEA_EXECUTOR_BUDGET_RATE=0.001
```

ALPHASEA_EXECUTOR_BUDGET_RATEは毎ラウンドの購入予算を、
ウォレットのMATIC残高に対する割合で指定するものです。
1日12ラウンドあるので、例えば0.001だと30日で最大30%減るくらいの設定です。
成績に寄与しない予測と自分の予測は購入しないので、予算を全て使わない場合もあります。
Predictorだけやる場合は、0を設定すれば購入しなくなります。
デフォルト値は0です。

alphasea-agentとredis(alphasea-agentが使う)を起動。

```bash
cd alphasea-agent
docker-compose -f docker-compose.yml up -d
```

## インスタンスBでalphasea-example-modelを動かす

[alphasea-example-model](https://github.com/alphasea-dapp/alphasea-example-model)を動かします。
Predictor(予測を売る人)をやるために必要です。

```bash
cd ~/
git clone https://github.com/alphasea-dapp/alphasea-example-model.git
```

[alphasea-example-model](https://github.com/alphasea-dapp/alphasea-example-model)
の手順にしたがって.envを作成。ALPHASEA_AGENT_BASE_URLにインスタンスAの内部IPを設定します

.envの例
```text
ALPHASEA_AGENT_BASE_URL=http://internal_ip:8070
ALPHASEA_MODEL_ID=my_model_id
ALPHASEA_MODEL_PATH=/app/data/example_model_rank.xz
```

ALPHASEA_MODEL_PATHは学習したモデルのパス(dockerコンテナ内でのパス)を指定します。
学習済みモデルファイルと学習方法は以下を見てください。

- [alphasea-example-model/data](https://github.com/alphasea-dapp/alphasea-example-model/tree/master/data)
- [alphasea-example-model/notebooks](https://github.com/alphasea-dapp/alphasea-example-model/tree/master/notebooks)

alphasea-example-modelを起動

```bash
docker-compose up -d
```

alphasea-example-modelは、
2時間ごとにインスタンスAのalphasea-agentに対して予測を投稿します。

## インスタンスBでalphasea-trade-botを動かす

[alphasea-trade-bot](https://github.com/alphasea-dapp/alphasea-trade-bot) を動かします。
Executor(予測を買ってCEXでトレードする人)をやるために必要です。
対応している取引所は[alphasea-trade-bot](https://github.com/alphasea-dapp/alphasea-trade-bot)を見てください。

```bash
cd ~/
git clone https://github.com/alphasea-dapp/alphasea-trade-bot.git
```

[alphasea-trade-bot#動かし方](https://github.com/alphasea-dapp/alphasea-trade-bot#%E5%8B%95%E3%81%8B%E3%81%97%E6%96%B9)
の手順にしたがって.envを作成。ALPHASEA_AGENT_BASE_URLにインスタンスAの内部IPを設定します

.envの例
```text
ALPHASEA_AGENT_BASE_URL=http://internal_ip:8070
上記の手順に記載のCCXT設定
```

alphasea-trade-botを起動

```bash
docker-compose up -d
```

alphasea-trade-botは、
CEXのPERPを、
インスタンスBのalphasea-agentから取得したメタモデルポジションに合わせるように、
リバランスし続けます。

## リーダーボードを見る

以下でAlphaSea (Polygon mainnet)のリーダーボードを見れます。

[AlphaSea Leaderboard (mainnet)](https://app.alphasea.io/)

ブロックチェーン上にあるデータを表示しているだけなので、
以下を動かせばローカルで同じものを見れます。

- [alphasea-ui](https://github.com/alphasea-dapp/alphasea-ui)
- [alphasea/subgraph](https://github.com/alphasea-dapp/alphasea)

## 次に何をすれば良い？

モデル改良の手引き(未執筆)

## FAQ

[@richmanbtc2](https://twitter.com/richmanbtc2)で質問してください。

都度加筆
