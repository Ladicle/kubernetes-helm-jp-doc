# Architecture

## Terms
Name    | Description
--------|------------
Chart   | k8sのインスタンスを作成するために必要にな情報群
Config  | `Relese`オブジェクトを作成するために、パッケージされた`Chart`に情報を組み込むための設定
Release | 実行中の指定した`Config`を組み込んだ`Chart`インスタンス

> [12FactorApp][1] に則って、Chart + Config = Relase

## The Purpose of Helm

Helmは`chart`と呼ばれるk8sパッケージを管理するためのツール。

## できること:
* `chart`をスクラッチで作成する
* `chart`をアーカイブしてtgzファイルにする
* k8sのクラスタへ`chart`のインストール/アンインストール
* Helmによってインストールされた`chart`の`release`管理

## Components
Helmは2つの主要コンポーネントを持っている。

## The Helm Client - コマンドラインクライアント
* ローカルの`chart`開発
* レポジトリの管理
* `Tiller`サーバとの通信
    * インストールする`chart`の送信
    * `Release`情報の取得
    * `Release`に対するアップグレードやアンインストールの要求

## The Tiller Server - Helmクライアントとk8s APIサーバのインタフェース
* Helmクライアントからのリクエスト受付
* `chart`と`configuration`を組み合わせた`release`のビルド
* `chart`をk8sへインストールし、`release`をトラックする
* k8sへ`chart`のアップグレードとアンインストールをする

簡潔に言うと、クライアントは`chart`の、サーバは`release`の管理を行っている。

## Inplementation
HelmクライアントはGoで書かれていて、Tillerサーバとの通信はgRPCが使用されている。
TillerサーバもまたGoで書かれていて、gRPCサーバの役目をもち、clientの接続を受けつけている。K8sのクライアントライブラリは、k8sとREST+JSONで通信している。(今のところ)
Tilerサーバの情報はk8sのConfigMapに保存されているので、サーバ自身はDBを必要としていない。

## Structure of the Code
- cmd/ : 個別のプログラム
- pkg/ : 共通ライブラリ
- _proto/hapi : 生のProtoBufファイル
- pkg/proto : `proto`から生成されたgoファイル
- scripts/ : k8sをローカル起動させる`local-cluster.sh`のような汎用スクリプト
- vendor/ : Glideによって管理されたGoの依存ライブラリ

> DockerイメージはLinuxバイナリのクロスコンパイルによってビルドされ、rootfsにあるファイルからDockerイメージが生成されます
