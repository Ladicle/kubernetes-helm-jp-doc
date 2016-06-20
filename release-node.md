# [ROAD MAP](https://github.com/kubernetes/helm/issues/694)

Alpha-1: [conplete!]

* Tiller内部にリリー用のインメモリストレージを組み込む
* k8sベースのhelmからtillerを経由したトンネルをはる
* `helm init`コマンドの実行時に
  * k8sのクラスタへTillerをインストール
  * ローカルの環境設定を行う
* `helm install CHART`コマンドを使ったchartのインストール
* `helm list`コマンドを使ったchartの一覧表示
* `helm get manifest, helm get values`コマンドによるrelease情報の取得
* `helm repo add/delete/update/list`コマンドを使用したレポジトリの管理
* `helm fetch, helm search`コマンドを使用したpackageの検索
* `helm create, helm package`を使ったchartパッケージの作成

Alpha-2: [Doing]

* values.tomlのグルーバル変数対応 #630
* レポジトリからchartのインストール対応 (helm install -v myvalues.toml kubernetes-charts/nginx-0.1.0)
* `helm lint`コマンドを使用したchartのテスト
* `helm serve`コマンドを使用した開発用のhelmパッケージの実行

Alpha-3:
* ConfigMapをベースとしたReleaseのストレージ
* `helm upgrade`コマンドを使用した基本的なReleaseのアップグレード
* 2.0.0に向けたnamespaceの方針を決める

Alpha-4:
* Releaseバックエンドの代替をサポートする(GCPとか)
 Ability to support alternate release backends (e.g. to GCP). #634参照
 
Alpha-N:
* Alphaレベルの機能を完成させる

Beta-1:
* Chartのフォーマットを決定する
* Go言語によるテンプレート表記を決定する
* Releaseのアップグレードを安定させる
* 操作方法をドキュメントにまとめる

Beta-2:
* kubernetes/chartsにサンプルとなるchartを追加する

Beta-N:
* すでに存在する機能をFIXさせる
* コミュニティで必要とされた機能を追加する

RC-1:
* Protoの定義をFIXする(fieldsの変更や削除をおこなわない)
* ComfigMapのストレージを安定化させる
* Tillerを安定化させる
* Helmを安定化させる
* kubernetes/chartsのchartsを動くものにする

RC-N:
* 非公式なリリースバージョンとして、見つかったバグなどの修正を行う

2.0.0 Final:
* Helmを安定化させ、テストやドキュメントを追加する
