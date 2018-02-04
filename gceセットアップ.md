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

kubectlがクラスタに接続できるように認証情報をダウンロードする。

`gcloud container clusters get-credentials usnode`

kubectlコマンドをインストールする

`gcloud components install kubectl`

## Google Container Builder

コンテナをビルドするのにローカルじゃなくてGoogle Container Builderを使う方法がある。ローカルにDockerが要らない。

ymlを使ってビルドするのが良いかも。

使えるリージョンはus,eu,asia。クラスタのリージョンに近いのを選ぶと良いが、リージョン名は同じではない点に注意。

https://cloud.google.com/container-registry/docs/pushing?hl=ja

```yaml
steps:
    - name: 'gcr.io/cloud-builders/docker'
      env: ['PROJECT_ROOT=usnode']
      args: ['build', '--tag=us.gcr.io/$PROJECT_ID/usnode/servertest', '.']
images: 
    - 'us.gcr.io/$PROJECT_ID/usnode/servertest'
```

```bash
$ gcloud container builds submit --config cloudbuild.yml .
```




