---
labels: [BOSH, 社内勉強会]
---

# BOSH Release概要

私の働いているチームでは日時での勉強会を行っている。だいたい週に1度ほど自分のターンが回ってくる。その勉強会での発表資料を兼ね、BOSHのReleaseについて解説しようと思う。

## Releaseとは何か

https://bosh.io/releases

> A release is a versioned collection of configuration properties, configuration templates, start up scripts, source code, binary artifacts, and anything else required to build and deploy software in a reproducible way

→ソフトウェアのデプロイに必要なもの一式を固めたもの。

後述のReleaseを定義するためのプロジェクトを作成し、プロジェクトをビルドすることで作成する。

## Releaseプロジェクト

Releaseを定義するにはYAMLファイルを中心としたプロジェクトを作成する。以下のコマンドでプロジェクトテンプレートを作成できる。

```
$ mkdir foo-release && cd foo-release
$ bosh init-release
```

以下のファイル・ディレクトリが生成される。

```
.
├── config/
│   ├── blobs.yml
│   └── final.yml
├── jobs/
├── packages/
└── src/
```

各ファイル・ディレクトリには次の構成要素に従った内容を記述し、`git commit`しておく。

## Releaseの構成要素

### Job

Releaseに含まれるサービス（アプリケーション）のこと。jobs/ディレクトリに起動・停止スクリプト及び、設定ファイルとして記述する。init.dやsystemctlでのserviceの概念が近い。

### Package

Releaseに含まれるパッケージ定義のこと。jobが利用するファイルの生成・インストールを行う。一般のOSでのrpm/aptパッケージなどが担う役割が近い。packages/ディレクトリにインストールスクリプトとして記述する。

### Blob

packageが利用するバイナリファイルのこと。blobstoreと呼ばれるデータベースに登録される。Releaseをビルドする際にblobstoreより取得する。このためReleaseをビルドする前にあらかじめ `bosh upload-blob` によりblobstoreにアップロードしておく。尚、blobstoreにはS3互換のオブジェクトデータベースが用いられることが多い。

### Source

packageが利用するソースコードやテキストファイルのこと。blobと異なりReleaseプロジェクト自身にコミットしておくことが一般的。

## Deploymentとの関係

Releaseの扱う構成管理の範囲は1VM内部に限定されている。一方、Deploymentの方がより上位の範囲を扱う。
具体的には、Releaseは、1VM内部で動くジョブ（プロセス）レベルの構成管理が中心である。一方、Deploymentは、VMインスタンス数や、VMに適用するイメージ、そしてVMにデプロイするReleaseなどVM
レベルの構成管理が中心である。

## Releaseをデプロイするまでの作業フロー

1. `bosh init-release`でプロジェクトテンプレート作成
2. プロジェクトを編集
3. 必要なblobを `bosh upload-blob` でblobstoreにアップロード
4. Releaseを `bosh create-release` でビルド
5. Deployment manifestにReleaseを指定する
6. `bosh deploy`

複雑なReleaseを作る場合、一気通貫で1~6を実施するのは現実的ではない。小さく作りながら上記を何サイクルも繰り返すのが良い。

## 参考資料
