---
title: "AlphaSea チュートリアル for eth testnet(ropsten)"
description: "AlphaSea is a decentralized marketplace for market alphas."
date: 2022-01-12T00:00:00+00:00
draft: false
---

AlphaSeaのtestnet(ropsten)に参加する方法。
作業メモ兼チュートリアルです。

## このチュートリアルでやること

1. alphasea-agentを動かす (2, 3を動かすために必要)
2. alphasea-example-modelを動かす (予測を売るために必要)
3. alphasea-trade-botを動かす (予測を買ってCEXでトレードするために必要)

注意

- このチュートリアルはethのtestnet(ropsten)で動かすので、ガス代や予測購入費用はかかりません
- alphasea-trade-botを動かすとCEXトレードで損失する可能性があります
- 自己責任
- 現状、バグで動きません
- 特に連絡無く修正していきます (コントラクトの再デプロイなど。[docker-compose-ropsten.yml](https://github.com/alphasea-dapp/alphasea-agent/blob/master/docker-compose-ropsten.yml)のALPHASEA_CONTRACT_ADDRESSが変更されていたら再デプロイされたと思ってください)

## Google Compute Engineインスタンスを作る

- マシンタイプ: e2-medium
- OS: Container optimized OS 93 LTS (docker version 20.10.0以上 [補足](https://qiita.com/skobaken/items/03a8b9d0e443745862ac))
- ディスク: 40GB SSD

このチュートリアル実行直後のリソース

- ディスク消費量: 6.7GB
- メモリ消費量: 1GB

以下、このインスタンス内で作業。

## docker-composeをインストール

以下に従い、docker-composeをインストール

[Running Docker Compose with Docker](https://cloud.google.com/community/tutorials/docker-compose-on-container-optimized-os)

## alphasea-agentを動かす

[alphasea-agent](https://github.com/alphasea-dapp/alphasea-agent) を動かします。

```bash
cd ~/
git clone https://github.com/alphasea-dapp/alphasea-agent.git
```

以下に従い秘密鍵を用意し、ウォレットにETHを入れる。

[alphasea-agent#秘密鍵](https://github.com/alphasea-dapp/alphasea-agent#%E7%A7%98%E5%AF%86%E9%8D%B5%E3%82%92%E7%94%A8%E6%84%8F)

alphasea-agentとgeth(ropsten用)を起動

```bash
cd alphasea-agent
docker-compose -f docker-compose-ropsten.yml up -d
```

## alphasea-example-modelを動かす

[alphasea-example-model](https://github.com/alphasea-dapp/alphasea-example-model)を動かします。

```bash
cd ~/
git clone https://github.com/alphasea-dapp/alphasea-example-model.git
```

[alphasea-example-model](https://github.com/alphasea-dapp/alphasea-example-model) 
の手順にしたがって.envを作成。

alphasea-example-modelを起動

```bash
docker-compose up -d
```

## alphasea-trade-botを動かす

[alphasea-trade-bot](https://github.com/alphasea-dapp/alphasea-trade-bot) を動かします。

```bash
cd ~/
git clone https://github.com/alphasea-dapp/alphasea-trade-bot.git
```

[alphasea-trade-bot#動かし方](https://github.com/alphasea-dapp/alphasea-trade-bot#%E5%8B%95%E3%81%8B%E3%81%97%E6%96%B9)
の手順にしたがって.envを作成。

alphasea-trade-botを起動

```bash
docker-compose up -d
```

## リーダーボードを見る

以下でAlphaSea testnet(ropsten)のリーダーボードを見れます。

[AlphaSea Leaderboard](https://alphasea-app-ropsten.netlify.app/)

以下を動かせばローカルで同じものを見れます。

- [alphasea-ui](https://github.com/alphasea-dapp/alphasea-ui)
- [alphasea](https://github.com/alphasea-dapp/alphasea)のthegraph

## FAQ

[@richmanbtc2](https://twitter.com/richmanbtc2)で質問してください。

都度加筆