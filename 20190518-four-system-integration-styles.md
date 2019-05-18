---
labels: [ソフトウェア開発]
---

# 4つのシステム連携手法

システム連携はシステム開発における基本である。しかし、連携ミスや、データ不整合、複雑化など幾多の課題が現れる難しい領域でもある。
大小様々なシステムを開発するにあたり、システム連携の設計に苦心することが多い。

システム連携の王道の形つまり、パターンにはどのようなものがあるのか改めて知っておきたい。
Martin Fowlerの[Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/patterns/messaging/IntegrationStylesIntro.html)によれば、システム連携には4つの手法があるとのことだ。Enterprise Integration Patternsの内容を踏まえ、各手法について自分なりの見解を述べてみる。

## 各手法

### 1. [File Transfer](https://www.enterpriseintegrationpatterns.com/patterns/messaging/FileTransferIntegration.html)

システム連携の最も基本的な形である。あるアプリケーションが出力するファイルを別のアプリケーションで読み込む。

これだけのことだが、ファイルを用いることで異なる言語で実装されているアプリケーション間の連携を実現できる。

ファイルのフォーマットには行ベースのフォーマットやXMLなどがあるが、最近のアプリケーションであればJSONやYAMLが多いだろう。

ただし、ファイルを直接出力し直接読み込むことから、アプリケーション間でディレクトリツリーを共有している必要がある。
NFSなどを使わない限り、アプリケーションが載るホストを超えた連携は不可能である。

昨今では、クラウドデザインパターンの[サイドカーパターン](https://docs.microsoft.com/ja-jp/azure/architecture/patterns/sidecar)と組み合わせることが多いと思う。ファイルを読み込むタイプのSidecarを用意し、メインのアプリケーションより出力されるファイルを介してサイドカーアプリケーションの連携する。

### 2. [Shared Database](https://www.enterpriseintegrationpatterns.com/patterns/messaging/SharedDataBaseIntegration.html)

ファイルの変わりにデータベースを共有する形である。あるアプリケーションがデータを追加・更新し、別のアプリケーションがそれを取り出すことで連携が可能となる。

データベースの方式にもよるが、MySQLなどサーバー方式のデータベースであれば、ネットワークを超えた連携を簡単に実装できる。

また、SQLなどのクエリ言語の使えるデータベースであれば、クエリ言語によるデータ取得ができ、データ送信・取得処理を特別に実装することなく、連携するデータのフィルタリングや結合を柔軟に行える。ただし、この柔軟さはそのまま本手法の弱点ともいえ、アプリケーション間の密結合を引き起こしやすい問題がある。

### 3. [Remote Procedure Invocation](https://www.enterpriseintegrationpatterns.com/patterns/messaging/EncapsulatedSynchronousIntegration.html)

多くの場合、HTTP/REST通信として見られる形である。gRPCによって実装する場合も増えてきている印象だ。Java RMIなど本手法のための専用技術も存在するが、現在ではあまり使われておらずHTTPのようなWeb由来の技術が優勢である。

Shared Databaseに比較したときの大きな違いとして、トリガーとして作用できる点がある。Shared Databaseの場合は、データ変更イベントの発生ををデータの改修側がただちに検知できないが、Remote Procedure Invocationではそれができる。したがって、ただデータを連携するだけではなく、データ変更時に必要な操作を連携先に依頼できる。

また、データベースの内容を公開するShared Databaseに比較して、データベースを隠す形となるので、システムのカプセル化がしやすく、システム間を疎結合に保ちやすい利点がある。

### 4. [Messaging](https://www.enterpriseintegrationpatterns.com/patterns/messaging/Messaging.html)

Remote Procedure Invocationと異なり、メッセージバスを介したメッセージ通信によってシステム連携する手法である。メッセージ通信は非同期で行われる。現実に用いられる技術としてRabbit MQやApache Kafkaなどがよく見られる。

メッセージバス及び、非同期通信の導入により、連携先システムの障害や、性能劣化の影響をただちに受けないようになり、連携しているコンポーネントの一部分の障害が全体へ波及することを防止できる。

メッセージバスの設定及び、受信側システムの設定により、メッセージのエニーキャストやブロードキャストを実現できるため、連携先システムの厳密なホストを知らずに済む利点もある。このため、システムを多重化している場合に、インスタンス数増減時の設定変更が不要であり、システムをスケールしやすい利点がある。

一方、Remote Procedure Invocationと異なり、メッセージに対するレスポンスを表現できない。特に連携先システムからのデータ取得を本手法で実装する場合、データ取得依頼メッセージ送信とその結果メッセージ受信の2つを実装する必要があり、やや実装コストが高い。また、メッセージバスを別途保守・運用する必要があるため、この点でもRemote Procedure Invocationよりお手軽度は下がる。

## 4つの手法の関係

Enterprise Integration Patternsでは、上記4つの手法のうち、後者ほどより先端的としており、Messagingに絞った詳細な解説が書かれているようだ。Messagingの詳細について追って調べていきたいと思う。
