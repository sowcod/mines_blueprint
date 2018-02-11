## GCP初期設定

Google Compute EngineとGoogle Kubernetes Engineのサービスを有効化

## gcloudコマンドインストール
gcloudコマンドのインストール

https://cloud.google.com/sdk/docs/quickstart-mac-os-x

sudoなしでインストールするので、展開先は自分のユーザーディレクトリ以下が望ましい？

.bashrcに、インストール中に表示されたパスをsourceコマンドと併せて設定。


プロジェクトIDを設定

`gcloud config set project (プロジェクトID)`


リージョンを設定する。

リージョンの一覧

https://cloud.google.com/compute/docs/regions-zones/?hl=ja

`gcloud config set comput/zone us-west1-a`

無料枠についてはこちら

https://cloud.google.com/free/docs/frequently-asked-questions?hl=ja


ログイン

`gcloud auth login`

サーバーで使用できるバージョンを見る

```
$ gcloud container get-server-config
Fetching server config for us-west1
defaultClusterVersion: 1.8.6-gke.0
defaultImageType: COS
validImageTypes:
- COS
- UBUNTU
validMasterVersions:
- 1.8.7-gke.0
- 1.8.6-gke.0
- 1.8.5-gke.0
validNodeVersions:
- 1.8.7-gke.0
- 1.8.6-gke.0
- 1.8.5-gke.0
```

ノードとクラスタを作る

`gcloud container clusters create usnode --cluster-version=1.8.7-gke.0 --machine-type=f1-micro`

Webの管理画面でクラスタを作った場合、kubectlがクラスタに接続できるように認証情報をダウンロードする。
上記のようにコマンドで作成した場合は不要。

`gcloud container clusters get-credentials usnode`

kubectlコマンドをインストールする

`gcloud components install kubectl`

## Google Container Builder

コンテナをビルドするのにローカルじゃなくてGoogle Container Builderを使う方法がある。ローカルにDockerが要らない。

ymlを使ってビルドするのが良いかも。

使えるリージョンはus,eu,asia。クラスタのリージョンに近いのを選ぶと良い。

ただしイメージが配置されるストレージバケットのストレージクラスがMulti-Regionalになるので、事実上有料サービスである。

https://cloud.google.com/container-registry/docs/pushing?hl=ja

```yaml
steps:
    - name: 'gcr.io/cloud-builders/docker'
      env: ['PROJECT_ROOT=usnode']
      args: ['build', '--tag=us.gcr.io/$PROJECT_ID/usnode/servertest', '.']
images: 
    - 'us.gcr.io/$PROJECT_ID/usnode/servertest'
```
※$PROJECT_IDは展開する。

```bash
$ gcloud container builds submit --config=cloudbuild.yml .
```

どうしても無料で済ませたい場合は、DockerHubなどの無料リポジトリにイメージをPushする。

## Podデプロイ
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: hello-world
spec:
    containers:
        - image: us.gcr.io/$PROJECT_ID/usnode/servertest
          imagePullPolicy: Always
          name: hello-world
```
※$PROJECT_IDは展開する。

`kubectl create -f pod.yaml`

クラスタに接続テストする時は下記コマンドでlocalhost:portからポートフォワードさせる。

`kubectl port-foward $POD_NAME localport:port`

このPodの例ではGoogle Container Registryのプライベートリポジトリから取得するものだが、
その他のサービスのプライベートリポジトリからイメージをpullする時は、`kubectl create secret docker-registry`コマンド等を
利用して認証情報を登録する。

DockerHubのパブリックリポジトリからのpullであれば、認証情報は不要。

## ConfigMap
Pod間で設定情報の共有ができる。
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: hello-world-config
data:
    hello-world.message: 'Hello!ConfigMap!'
```

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: hello-world
spec:
    containers:
      - image: us.gcr.io/$PROJECT_ID/usnode/servertest
        imagePullPolicy: Always
        name: hello-world
        env:
          - name: TESTENV
            valueFrom:
                configMapKeyRef:
                    name: hello-world-config
                    key: hello-world.message
```
これで、hello-worldコンテナで、$TESTENVの中身が'Hello!ConfigMap!'となる。

## ReplicaSet
kind: ReplicaSetでyamlを作る。

Podの数を常に一定に保つ機能。

## Deployment
kind: Deploymet。

yamlの中身はReplicaSetとほぼ同じ。ただし、kubectl updateで
反映する度に、前にupdateしたDeploymentを覚えた上で、最新のものに反映される。

本番移行後に問題が発生した時に取り急ぎもとに戻すといったような使い方ができる。

## Service

クラスタの外側に向いているもの。

概念はともかくとして、type:LoadBalancerのServiceを用いて、DeploymentもしくはReplicaSetの各Podに、外部からの接続が出来るようにする

## NodePool

特定のPodだけは専用仕様ノードで、というような事をするには、NodePoolを作成する事で、異なるマシンタイプのノードを利用できる。

ノードのリージョンも指定できるが、同じゾーン内という制限がある。例えばUSとJPをまたぐクラスタ構成は作ることができない。
