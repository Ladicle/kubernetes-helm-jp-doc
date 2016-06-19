# Quickstart Guid
## helmのインストール
1. Set property. 
```
export GO15VENDOREXPERIMENT=1
```
2. Clone helm repository.
```
$ git clone github.com/kubernetes/helm k8s.io/helm
```
> pathが変わったので注意

3. Build binary.
```
$ make bootstrap build
```
4. You will now have two binaries :)
```
$ ls bin
helm
tiller
```

## Chartをインストールする

Chartのレポジトリを登録するには`helm repo add`コマンドを実行します。

```bash
$ helm repo add kubernetes-charts http://storage.googleapis.com/kubernetes-charts
kubernetes-charts has been added to your repositories
```

登録済みのレポジトリは`helm repo list`コマンドで確認することができます。

```bash
$ helm repo list
NAME                    URL
local                   http://localhost:8879/charts
kubernetes-charts       http://storage.googleapis.com/kubernetes-charts
```

> 本家のドキュメントにはまだレポジトリ導入についてない
> また、このレポジトリはまだ正式なものでないので注意

登録済みのChartをインストールするには、
`helm install`コマンドを実行します。

```bash
$ helm install nginx-1.0.0
Released smiling-preguin
```

上記の例では、`nginx`のリリースされたChartです。
新しいリリースの名前が`smiling-penguin`です。

## Releaseについて

releaseについて情報を得るには、`helm status`コマンドを実行します

```bash
$ helm status smilling-penguin
Status: DEPLOYED
```

## Releaseをアンインストールする

Releaseをアンインストールするには`helm delete`コマンドを実行します

```bash
$ helm delete smiling-peguin
Status: DELETED
```

`smiling-peguin`はKubernetesからアンインストールされます。しかし、Releaseのステータスは取得することができます。

```bash
$ helm status smiling-penguin
Removed smilling-penguin
```

## ヘルプを読む

Helmのコマンドを調べるには、`helm help`コマンドもしくは`-h`フラグが使えます。

```bash
$ helm get -h
```
