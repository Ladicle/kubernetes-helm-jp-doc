# Charts

Helmは`charts`と呼ばれるフォーマットパックを使います。`chart`はKubernetesリソースの関連するファイルを集めたものです。`chart`は単体シンプルなものをデプロイするために使うことができ、簡単なものー例えばmemcachedの`Pod`や、何か複雑なHTTPサーバ, データベース, キャッシュなどのフルWebアプリケーションスタックなどです。
作成したChartsは特定のディレクトリツリーにファイルとして配置し、デプロイするためのバージョン付きアーカイブに圧縮することができます。

## Chartファイルの構成

Chartはディレクトリの中のファイル集合によって構成されています。ディレクトリの名前は(バージョン情報を含まない)chartの名前です。そのため、chartはWordpressのchartは`wordpress/`ディレクトリに配置されます。

ディレクトリの中身は、このような構成をとっています。
```
wordpress/
  Chart.yaml   # Chartsに関する情報をYAMLファイルにまとめてあります
  LICENSE      # オプション: 素のテキストファイルに、chartのライセンス情報を記述しています
  README.md    # オプション: 人が読みやすいようなREADMEファイルです
  values.yaml  # このchartのデフォルト値が設定されています
  charts/      # オプション: このディレクトリは、chartの二依存するchartsを配置します 
  templates/   # オプション: このディレクトリは、テンプレートを配置し、valuesと組み合わせてk8sのmanifestファイルを生成します
```

Helmは上記以外のファイルを読み込みません。


## The Chart.yaml FIle

`chart.yaml`ファイルはchartのために必須なものです。以下のようなフィールドを含みます:

```
name: The name of the chart(required)
version: A SemVer 2 version(required)
description: A single-sentence description of this project (optional)
keywords:
  - A list of keywords about this project (optional)
home: The URL of this project's home page (optional)
sources:
  - A list of URLs to source code for this project (optional)
maintainers: # (optional)
  - name: The maintainer's name (required for each maintainer)
    email: The maintainer's email (optional for each maintainer)
engine: gotpl # The name of the template engine (optional, defaults to gotpl)
```

もし、HelmClassicようのフォーマットで似たような`Chart.yaml`を持っていたら、`dependencies`のフィールドを削除すれば転用することができます。新しいChartのフォーマットでは`charts/`ディレクトリ内に依存chartsを配置するようになっています。

他のフィールドについては無視されます。

## Charts and Versioning

各Chartはバージョン情報を保持しています。バージョニングは[SemVer 2](http://semver.org/lang/ja)の仕様を使用しています。
Helm Classicとは異なり、Kubernetes Helmはバージョンをリリースの基準としています。
レポジトリ内のパッケージは、名前とバージョンで識別されています。

例えば、`nginx` chartのバージョンフィールドに `version: 1.2.3` と設定されていたとすると、名称は以下のようになります。

```
nginx-1.2.3.tgz
```

また、SemVer2 では`version: 1.2.3-aplpha.1+ef365`のような名称もサポートしています。
しかし、SemVerではないバージョニングはこのシステムでは無効です。

> **NOTE**
> Helm Classic と Deployment Managerは共にGithubからchartを持ってくることを前提に作られています。
> しかし、Kubernetes HelmではGitHubやGitが必須要件ではありません。
> そのため、バージョニングにGitのSHAsの値をを使用していません。

`Chart.yaml`のバージョンフィールドは、CLIやTiller serverなどいろいろなHelmのツールで使用されます。
packageを生成するとき、`helm package`コマンドは`Chart.yaml`のバージョンフィールドの値を取得し、パッケージ名に使用します。
システムはchartパッケージ名のバージョン情報が`Chart.yaml`のバージョンフィールドと一致していることを前提に動作します。
もしこの値が異なると、エラーが発生します。

## Chartの依存関係

Helmでは、一つのcharに対して複数のchartに対して依存関係を持たせることができます。
これらの依存関係は処理しやすいように依存関係にある`charts`ディレクトリの中にコピーされます。

> **NOTE:** Helm Classicにあった`Chart.yaml`の`dependencies:` フィールドは削除されました

例として、Apacheに依存しているWordpressのchartを以下に示しています。
Apacheの(正しいバージョンの)chartはWordpress chartの`charts/`ディレクトリに配置されていることがわかります。

```
wordpress:
  Chart.yaml
  # ...
  charts/
    apache/
      Chart.yaml
      # ...
    mysql/
      Chart.yaml
      # ...
```

上記の例のように、WOrdpressのchartはApacheとMySQLのchartを`charts/`に保持し、処理対象となります。

> **TIP:** `charts/`ディレクトリ中の依存関係を解消するために`helm fetch`コマンドを使用することができます

## テンプレートと値

デフォルトではHelm ChartのテンプレートはGoのテンプレート記法を使用しますが、アドオンとして[sprig](https://github.com/Masterminds/sprig)が組み込まれているので追加で50個ほどの昨日を使用することができます。(将来的には、HelmではPythonのJinjaのような他のテンプレートエンジンもサポートする予定です。)

すべてのテンプレートファイルはchartの`templates/`フォルダに配置されます。
Helmがchartを生成するとき、このディレクトリにあるすべてのファイルがテンプレートエンジンによって処理されます。

テンプレートに値を組み込む方法は2通りあります:
* Chartの開発者は`values.yaml`と呼ばれるchartの中にあるファイルにデフォルト値を定義することができます。
* ChartのユーザはYAMLファイルに組み込む値を定義し、`helm install`コマンドを実行するときに指定することができます

ユーザが値を指定するとき、chartの`values.yaml`ファイルを上書きする形になります。

### テンプレートファイル

テンプレートファイルは、以下のようにGoのテンプレートの標準的な変換がおこなわれます。
テンプレート例のはいるは、このような形になります。

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: deis-database
  namespace: deis
  labels:
    heritage: deis
spec:
  replicas: 1
  selector:
    app: deis-database
  template:
    metadata:
      labels:
        app: deis-database
    spec:
      serviceAccount: deis-database
      containers:
        - name: deis-database
          image: {{.imageRegistry}}/postgres:{{.dockerTag}}
          imagePullPolicy: {{.pullPolicy}}
          ports:
            - containerPort: 5432
          env:
            - name: DATABASE_STORAGE
              value: {{default "minio" .storage}}
```

--- 

TODO あとでやる


The above example, based loosely on [https://github.com/deis/charts](https://github.com/deis/charts), is a template for a Kubernetes replication controller.
It can use the following four template values:

- `imageRegistry`: The source registry for the Docker image.
- `dockerTag`: The tag for the docker image.
- `pullPolicy`: The Kubernetes pull policy.
- `storage`: The storage backend, whose default is set to `"minio"`

All of these values are defined by the template author. Helm does not
require or dictate parameters.

### Predefined Values

The following values are pre-defined, are available to every template, and
cannot be overridden. As with all values, the names are _case
sensitive_.

- `Release.Name`: The name of the release (not the chart)
- `Release.Time`: The time the chart release was last updated. This will
  match the `Last Released` time on a Release object.
- `Release.Namespace`: The namespace the chart was released to.
- `Release.Service`: The service that conducted the release. Usually
  this is `Tiller`.
- `Chart`: The contents of the `Chart.yaml`. Thus, the chart version is
  obtainable as `Chart.Version` and the maintainers are in
  `Chart.Maintainers`.

**NOTE:** Any unknown Chart.yaml fields will be dropped. They will not
be accessible inside of the `Chart` object. Thus, Chart.yaml cannot be
used to pass arbitrarily structured data into the template.

### Values files

Considering the template in the previous section, a `values.yaml` file
that supplies the necessary values would look like this:

```yaml
imageRegistry: "quay.io/deis"
dockerTag: "latest"
pullPolicy: "alwaysPull"
storage: "s3"
```

A values file is formatted in YAML. A chart may include a default
`values.yaml` file. The Helm install command allows a user to override
values by supplying additional YAML values:

```console
$ helm install --values=myvals.yaml wordpress
```

When values are passed in this way, they will be merged into the default
values file. For example, consider a `myvals.yaml` file that looks like
this:

```yaml
storage: "gcs"
```

When this is merged with the `values.yaml` in the chart, the resulting
generated content will be:

```yaml
imageRegistry: "quay.io/deis"
dockerTag: "latest"
pullPolicy: "alwaysPull"
storage: "gcs"
```

Note that only the last field was overridden.

**NOTE:** The default values file included inside of a chart _must_ be named
`values.yaml`. But files specified on the command line can be named
anything.

### Scope, Dependencies, and Values

Values files can declare values for the top-level chart, as well as for
any of the charts that are included in that chart's `charts/` directory.
Or, to phrase it differently, a values file can supply values to the
chart as well as to any of its dependencies. For example, the
demonstration Wordpress chart above has both `mysql` and `apache` as
dependencies. The values file could supply values to all of these
components:

```yaml
title: "My Wordpress Site" # Sent to the Wordpress template

mysql:
  max_connections: 100 # Sent to MySQL
  password: "secret"

apache:
  port: 8080 # Passed to Apache
```

Charts at a higher level have access to all of the variables defined
beneath. So the wordpress chart can access `.mysql.password`. But lower
level charts cannot access things in parent charts, so MySQL will not be
able to access the `title` property. Nor, for that matter, can it access
`.apache.port`.

Values are namespaced, but namespaces are pruned. So for the Wordpress
chart, it can access the MySQL password field as `.mysql.password`. But
for the MySQL chart, the scope of the values has been reduced and the
namespace prefix removed, so it will see the password field simply as
`.password`.

#### Global Values

As of 2.0.0-Alpha.2, Helm supports sspecial "global" value. Consider
this modified version of the previous exampl:

```yaml
title: "My Wordpress Site" # Sent to the Wordpress template

global:
  app: MyWordpress

mysql:
  max_connections: 100 # Sent to MySQL
  password: "secret"

apache:
  port: 8080 # Passed to Apache
```

The above adds a `global` section with the value `app: MyWordpress`.
This value is available to _all_ charts as `.global.app`.

For example, the `mysql` templates may access `app` as `{{.global.app}}`, and
so can the `apache` chart. Effectively, the values file above is
regenerated like this:

```yaml
title: "My Wordpress Site" # Sent to the Wordpress template

global:
  app: MyWordpress

mysql:
  global:
    app: MyWordpress
  max_connections: 100 # Sent to MySQL
  password: "secret"

apache:
  global:
    app: MyWordpress
  port: 8080 # Passed to Apache
```

This provides a way of sharing one top-level variable with all
subcharts, which is useful for things like setting `metadata` properties
like labels.

If a subchart declares a global variable, that global will be passed
_downward_ (to the subchart's subcharts), but not _upward_ to the parent
chart. There is no way for a subchart to influence the values of the
parent chart.

_Global sections are restricted to only simple key/value pairs. They do
not support nesting._

For example, the following is **illegal** and will not work:

```yaml
global:
  foo:  # It is illegal to nest an object inside of global.
    bar: baz
```

### References

When it comes to writing templates and values files, there are several
standard references that will help you out.

- [Go templates](https://godoc.org/text/template)
- [Extra template functions](https://godoc.org/github.com/Masterminds/sprig)
- [The YAML format]()

## Using Helm to Manage Charts

The `helm` tool has several commands for working with charts.

It can create a new chart for you:

```console
$ helm create mychart
Created mychart/
```

Once you have edited a chart, `helm` can package it into a chart archive
for you:

```console
$ helm package mychart
Archived mychart-0.1.-.tgz
```

You can also use `helm` to help you find issues with your chart's
formatting or information:

```console
$ helm lint mychart
No issues found
```

## Chart Repositories

A _chart repository_ is an HTTP server that houses one or more packaged
charts. While `helm` can be used to manage local chart directories, when
it comes to sharing charts, the preferred mechanism is a chart
repository.

Any HTTP server that can serve YAML files and tar files and can answer
GET requests can be used as a repository server.

Helm comes with built-in package server for developer testing (`helm
serve`). The Helm team has tested other servers, including Google Cloud
Storage with website mode enabled, and S3 with website mode enabled.

A repository is characterized primarily by the presence of a special
file called `index.yaml` that has a list of all of the packages supplied
by the repository, together with metadata that allows retrieving and
verifying those packages.

On the client side, repositories are managed with the `helm repo`
commands. However, Helm does not provide tools for uploading charts to
remote repository servers. This is because doing so would add
substantial requirements to an implementing server, and thus raise the
barrier for setting up a Repository.

