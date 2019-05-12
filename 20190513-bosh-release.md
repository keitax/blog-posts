---
labels: [BOSH, 社内勉強会]
---

# BOSH Release概要

私の働いているチームでは、20分ほどの技術勉強会を日時で行っている。持ち回り制で週に1度ほど自分のターンが回ってくる。その勉強会での発表資料を兼ね、BOSHのReleaseについて解説しようと思う。

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

Releaseは以下の各構成要素より成り立つ。

### Job

Releaseに含まれるサービス（アプリケーション）のこと。jobs/ディレクトリに起動・停止・監視スクリプト及び、設定ファイルから定義される。init.dやsystemctlでのserviceの概念が近い。

### Package

Releaseに含まれるパッケージ定義のこと。Jobが利用するファイルの生成・インストールを行う。一般のOSでのrpm/aptパッケージなどが担う役割が近い。packages/ディレクトリにインストールスクリプトとして記述する。

### Blob

Packageが利用するバイナリファイルのこと。blobstoreと呼ばれるデータベースに登録される。Releaseをビルドする際にblobstoreより取得する。このためReleaseをビルドする前にあらかじめ`bosh upload-blob`によりblobstoreにアップロードしておく。尚、blobstoreにはS3互換のオブジェクトストレージが用いられる。

### Source

packageが利用するソースコードやテキストファイルのこと。blobと異なりReleaseプロジェクト自身にコミットしておくのが一般的。

## Releaseの種類
 
production環境にデプロイするためのReleaseはFinal Releaseと呼ぶ。一方開発用途としてDev Releaseが存在する。Dev ReleaseのFinal Releaseとの違いは以下である。

- `bosh create-release`ではなく`bosh create-release --force`で作成する
- Releaseプロジェクトがuncommittedでも`create-release`できる
- blobを`bosh add-blob`しておくことでblobstoreにアップロードせずにdeploy可能

## ReleaseとDeploymentの違い

Releaseの扱う構成管理の範囲は1VM内部に限定されている。一方、Deploymentはより上位の範囲を扱う。
具体的には、Releaseは、1VM内部で動くサービス（プロセス）レベルの構成管理が中心である。一方、Deploymentは、VMインスタンス数や、VMに適用するイメージ、そしてVMにデプロイするReleaseなどVM
レベルでの構成管理が中心である。

## Releaseをデプロイするまでの作業フロー

大まかな作業は以下である。

1. `bosh init-release`でプロジェクトテンプレート作成
2. プロジェクトを編集
3. 必要なblobを`bosh upload-blob`でblobstoreにアップロード
4. Releaseを`bosh create-release`でビルド
5. Deployment manifestにReleaseを指定する
6. `bosh deploy`

複雑なReleaseを作る場合、一気通貫で1~6を実施するのは現実的でない。小さく作りながら上記を何サイクルも繰り返すのが良い。またRelease開発時は効率化のため、前述のDev Release作成が推奨される。


## Releaseのデータフロー

![](https://dl.dropboxusercontent.com/s/qm455ir88rngdws/20190513-bosh-release.png?dl=0)

## 参考資料

- [Creating a Release - Cloud Foundry BOSH](https://bosh.io/docs/create-release/)
