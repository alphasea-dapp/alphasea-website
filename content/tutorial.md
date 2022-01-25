---
title: "AlphaSea チュートリアル"
description: "AlphaSea is a decentralized marketplace for market alphas."
date: 2022-01-24T00:00:00+00:00
draft: false
---

AlphaSeaのチュートリアルです。

- [AlphaSeaとは？](/introduction)
- [AlphaSeaの仕組み](/how-it-works)

## このチュートリアルでやること

やること

- alphasea-agentを動かす
- alphasea-example-modelを動かす (Predictorをやる場合のみ)
- alphasea-trade-botを動かす (Executorをやる場合のみ)

実現されること

- 毎ラウンド予測を売る (Predictor)
- メタモデルポジションに従って仮想通貨取引所で自動トレード (Executor)

注意

- このチュートリアルはPolygonのmainnetで動かすので、実際にガス代や予測購入費用などがかかります
- alphasea-trade-botを動かすと自動トレードで損失する可能性があります
- バグも含めて自己責任

\* PredictorとExecutorについては[AlphaSeaの仕組み](/how-it-works)参照

## 1. Google Compute Engineインスタンスを作る

alphasea-agent, alphasea-example-model, alphasea-trade-botを動かすためのサーバーを用意します。
Google Compute Engineで以下の構成のインスタンスを2つ作ってください。
以後、インスタンスA, Bと呼びます。

- マシンタイプ: e2-medium
- OS: Container optimized OS 93 LTS (docker version 20.10.0以上 [補足](https://qiita.com/skobaken/items/03a8b9d0e443745862ac))
- ディスク: 40GB

インスタンスと動かすプログラムの対応は以下のようになります。

|インスタンス|動かすプログラム|
|:-:|:-:|
|A|alphasea-agent|
|B|alphasea-example-model, alphasea-trade-bot|

インスタンスAはインスタンスBからアクセスされるので、
再起動してもIPアドレスが変わらないように、
[静的内部 IP アドレスの予約](https://cloud.google.com/compute/docs/ip-addresses/reserve-static-internal-ip-address)
をすると良いかもしれません。

Google Compute Engineの代わりにAWSや他のVPSでも良いです。
ただし、dockerとdocker-composeが動く必要があります。

## 2. docker-composeをインストール

インスタンスA, Bそれぞれで以下のコマンドを実行し、docker-composeをインストールします。
Container optimized OSを使った場合、dockerはすでにインストール済みなので、docker-composeのみでokです。

```bash
echo alias docker-compose="'"'docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v "$PWD:$PWD" \
    -w="$PWD" \
    docker/compose:1.24.0'"'" >> ~/.bashrc
source ~/.bashrc
```

source: [Running Docker Compose with Docker](https://cloud.google.com/community/tutorials/docker-compose-on-container-optimized-os)

以下のコマンドを実行し、バージョン情報が表示されたら正常にインストールできています。

```bash
docker-compose version
```

## 3. インスタンスAでalphasea-agentを動かす

インスタンスAで
[alphasea-agent](https://github.com/alphasea-dapp/alphasea-agent) を動かします。

alphasea-agentは、AlphaSeaスマートコントラクトを、 シンプルなインターフェースで扱えるようにするためのHTTPサーバーです。
後で出てくるalphasea-example-modelとalphasea-trade-botを動かすために必要です。

まず、以下のコマンドを実行し、alphasea-agentリポジトリをホームディレクトリにクローンします。

```bash
cd ~/
git clone https://github.com/alphasea-dapp/alphasea-agent.git
```

以下のリンク先の手順に従い、alphasea-agentを動かします。
リンク先ではウォレット作成、MATIC入金、.envファイル作成、起動を行います。

[alphasea-agent #agentの動かし方](https://github.com/alphasea-dapp/alphasea-agent#agent%E3%81%AE%E5%8B%95%E3%81%8B%E3%81%97%E6%96%B9)

## 4. インスタンスBでalphasea-example-modelを動かす

インスタンスBで
[alphasea-example-model](https://github.com/alphasea-dapp/alphasea-example-model) を動かします。

alphasea-example-modelは alphasea-agent に対して毎ラウンド、予測を投稿するプログラムです。
Predictorをやるために必要です。

まず、以下のコマンドを実行し、alphasea-example-modelリポジトリをホームディレクトリにクローンします。

```bash
cd ~/
git clone https://github.com/alphasea-dapp/alphasea-example-model.git
```

以下のリンク先の手順に従い、alphasea-example-modelを動かします。
リンク先では.envファイル作成、起動を行います。
.envに設定するALPHASEA_AGENT_BASE_URLには、インスタンスAの内部IPアドレスを設定してください。

[alphasea-example-model #動かし方](https://github.com/alphasea-dapp/alphasea-example-model#%E5%8B%95%E3%81%8B%E3%81%97%E6%96%B9)

## 5. インスタンスBでalphasea-trade-botを動かす

インスタンスBで
[alphasea-trade-bot](https://github.com/alphasea-dapp/alphasea-trade-bot) を動かします。

alphasea-trade-botは、定期的にalphasea-agent からメタモデルポジションを取得し、仮想通貨取引所の無期限先物(Perpetual)のポジションを自動でリバランスするボットです。
対応している取引所はFTXとBinanceです。
Executorをやるために必要です。

まず、以下のコマンドを実行し、alphasea-trade-botリポジトリをホームディレクトリにクローンします。

```bash
cd ~/
git clone https://github.com/alphasea-dapp/alphasea-trade-bot.git
```

以下のリンク先の手順に従い、alphasea-trade-botを動かします。
リンク先では.envファイル作成、起動を行います。
.envに設定するALPHASEA_AGENT_BASE_URLには、インスタンスAの内部IPアドレスを設定してください。

[alphasea-trade-bot#動かし方](https://github.com/alphasea-dapp/alphasea-trade-bot#%E5%8B%95%E3%81%8B%E3%81%97%E6%96%B9)

## 6. リーダーボード確認

以下のリンクでAlphaSea (Polygon mainnet)のリーダーボードを見れます。
ブロックチェーン上にあるデータを表示しています。

[AlphaSea Leaderboard (mainnet)](https://app.alphasea.io/)

alphasea-example-modelのセットアップが正常に完了していれば、
2時間以内にリーダーボード上に自分のモデルが表示されます。

もし、表示されない場合は、各プログラムのログを確認してみてください。
以下のコマンドでログを確認できます。

```bash
docker-compose logs
```

以上でセットアップは完了です。

## 次に何をすれば良い？

Predictorをやる場合は、
以下に従ってモデルを改良します。

[alphasea-example-model#モデル改良](https://github.com/alphasea-dapp/alphasea-example-model#%E3%83%A2%E3%83%87%E3%83%AB%E6%94%B9%E8%89%AF)

Executorだけやる場合は、
alphasea-trade-botがトレードするのを待つだけなので、
特にやることは無いです。

## FAQ

ツイッターかgithub issueで質問してください。

- [@richmanbtc2](https://twitter.com/richmanbtc2)
- [alphasea-agent](https://github.com/alphasea-dapp/alphasea-agent)
- [alphasea-example-model](https://github.com/alphasea-dapp/alphasea-example-model)
- [alphasea-trade-bot](https://github.com/alphasea-dapp/alphasea-trade-bot)
