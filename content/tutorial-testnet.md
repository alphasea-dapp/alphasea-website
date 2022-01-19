---
title: "AlphaSea チュートリアル for polygon testnet(mumbai)"
description: "AlphaSea is a decentralized marketplace for market alphas."
date: 2022-01-12T00:00:00+00:00
draft: false
---

AlphaSeaのtestnet(mumbai)に参加する方法。
作業メモ兼チュートリアルです。
[AlphaSeaの仕組み](/how-it-works)も見ると理解しやすいです。

## このチュートリアルでやること

1. alphasea-agentを動かす (2, 3を動かすために必要)
2. alphasea-example-modelを動かす (Predictorをやるために必要)
3. alphasea-trade-botを動かす (Executorをやるために必要)

注意

- このチュートリアルはpolygonのtestnet(mumbai)で動かすので、ガス代や予測購入費用には、faucetサイトから無料で取得したtestnet用のETHを使えます
- alphasea-trade-botを動かすとCEXトレードで損失する可能性があります
- 一応、動いていますが、作ったばかりなのでバグがあるかもしれません
- 特に連絡無く修正していきます (コントラクトの再デプロイなど。[docker-compose-mumbai.yml](https://github.com/alphasea-dapp/alphasea-agent/blob/master/docker-compose-mumbai.yml)のALPHASEA_CONTRACT_ADDRESSが変更されていたら再デプロイされたと思ってください)
- 自己責任

## Google Compute Engineインスタンスを作る

以下の構成のインスタンスを2つ作ります。
インスタンスA, Bと呼びます。

- マシンタイプ: e2-medium
- OS: Container optimized OS 93 LTS (docker version 20.10.0以上 [補足](https://qiita.com/skobaken/items/03a8b9d0e443745862ac))
- ディスク: 40GB SSD

### 補足

このチュートリアル実行直後のリソース

- ディスク消費量: 6.7GB
- メモリ消費量: 1GB

インスタンスを分ける理由は、
Google Compute Engineだとそのままだと、
containerからhostにアクセスできなかったため。
ファイアウォールとかをいじればできるのかもしれないが、
ミスると嫌なので2つ作りました。
この辺は各自好きな方法でやれば良いと思います。

ディスクは10GBだと少し足りない気がします。

## docker-composeをインストール

以下に従い、全てのインスタンスにdocker-composeをインストール

[Running Docker Compose with Docker](https://cloud.google.com/community/tutorials/docker-compose-on-container-optimized-os)

## インスタンスAでalphasea-agentを動かす

[alphasea-agent](https://github.com/alphasea-dapp/alphasea-agent) を動かします。
Predictor, Executorをやるために必要です。

```bash
cd ~/
git clone https://github.com/alphasea-dapp/alphasea-agent.git
```

以下に従い秘密鍵を用意し、ウォレットにMATIC(testnet用)を入れる。

[alphasea-agent#秘密鍵](https://github.com/alphasea-dapp/alphasea-agent#%E7%A7%98%E5%AF%86%E9%8D%B5%E3%82%92%E7%94%A8%E6%84%8F)

alphasea-agentとgeth(ropsten用)を起動

```bash
cd alphasea-agent
docker-compose -f docker-compose-mumbai.yml up -d
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
```

alphasea-example-modelを起動

```bash
docker-compose up -d
```

## インスタンスBでalphasea-trade-botを動かす

[alphasea-trade-bot](https://github.com/alphasea-dapp/alphasea-trade-bot) を動かします。
Executor(予測を買ってCEXでトレードする人)をやるために必要です。

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

## リーダーボードを見る

以下でAlphaSea testnet(mumbai)のリーダーボードを見れます。

[AlphaSea Leaderboard (mumbai)](https://alphasea-app-mumbai.netlify.app/)

ブロックチェーン上にあるデータを表示しているだけです。
以下を動かせばローカルで同じものを見れます。

- [alphasea-ui](https://github.com/alphasea-dapp/alphasea-ui)
- [alphasea](https://github.com/alphasea-dapp/alphasea)のthegraph

## FAQ

[@richmanbtc2](https://twitter.com/richmanbtc2)で質問してください。

都度加筆
