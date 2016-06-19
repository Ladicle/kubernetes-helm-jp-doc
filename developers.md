## Charts and Developers

# Versioning Guide

HelmとTillerの開発環境のセットアップ方法

## 必要なもの
- Go 1.6.0 or later
- Glide 0.10.2 or later
- kubectl 1.2 or later
- A Kubernetes cluster (optional)
- The gRPC toolchain

## HelmとTillerのビルド

以下のコマンドでHelmとTillerがビルドされます

```bash
make bootstrap build
```

## テストの実行方法

```bash
make test
```

## ローカルでHelmとTillerを実行する

`bin/helm`と`bin/tiller`を使うとローカルでHelmとTIllerが実行できます。

* HelmとTillerはOSXとAlpineを含むおおかたのLinuxで動きます
* Tillerはkubernetesクラスタにアクセスするので、kubeのコンフィグファイルを設定して`kubectl`が正常に動いている必要があります


## gRPCとProtobuf

HelmとTillerはgRPCを使って通信しています。gRPCを使うには以下のものが必要になります。

* protobufファイルをコンパイルするための`protoc`をインストールする。(リリースは[こちら][2])
* protocようのGoプラグインをインストールする。
  `go get -u github.com/golang/protobuf/protoc-gen-go`

protobufは3.x代(`protoc --version`)で、Goのプラグインは最新のものが必要です。

gRPCとProtoBufのインデント設計は、Goのフォーマットと一致しています。通常、プロトコルバッファはタブベースのインデントを使い、RPCの宣言にはGoの関数定義と同じスタイルになっています。

## Helm API (HAPI)

APIレイヤでgRPCを使っています。pkg/proto/hapi`を見るとわかるようにGoのコードで生成され、プロトコルバッファのための`_proto`が定義されています。

protobufからGoのファイルを生成するには、`make protoc`を実行します。

## Docker Image

Dockerイメージをビルドするには、`make docker-build`を使用します。

## ロカールでクラスタを動かす

ローカルでテストを実行する時、`scripts/local-cluster.sh`を使うとDockerコンテナの中でKubernetesが実行されます。OSXの場合は、docker-machineが動いている必要があります。

## Contributionガイド

contributionsを歓迎しています。このプロジェクトでは
a. コードのクウォリティを保つため
b. プロジェクトを安定させるため
c. contributionをオープンソースの決まりに乗っ取るため
幾つかのガイドラインを設定しています。

https://k8s.io/helm/blob/master/CONTRIBUTING.md
https://github.com/deis/workflow/blob/master/CONTRIBUTING.md
https://github.com/deis/workflow/blob/master/src/contributing/submitting-a-pull-request.md

また、contributorはCNCF/GoogleのCLAに同意する必要があります。
