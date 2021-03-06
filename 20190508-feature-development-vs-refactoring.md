---
labels: [ソフトウェア開発]
---

# 機能開発 v.s. リファクタリング

ソフトウェアを開発していると、通常の機能開発と、設計やコードを綺麗にするリファクタリングをどのようにするか迷うことがある。
この迷いについて自分の考えをまとめてみたい。

## 言葉の定義

「機能開発」と「リファクタリング」を定義を以下とする。

- 機能開発：ユーザー価値の観点でプラスになるが、開発者生産性の観点で変化のない開発
- リファクタリング：ユーザー価値の観点で変化がないが、開発者生産性の観点でプラスになる開発

以上に当てはまらない変化をもたらす開発もあるが、多くの場合は以上のどちらかにあたると思う。本記事では上記のみを考える。

## 何に迷うのか

私は以下の点に迷うことが多い。

1. どれぐらいリファクタリングに工数を裂くか
2. リファクタリングに着手してから機能開発するか、機能開発をしてからリファクタリングに着手するか

特に1については、多くの開発者の迷いどころではないだろうか。2についても同感の人がいるかもしれない。順番に考えていきたい。

### どれぐらいリファクタリングに工数を裂くか

結局はケースバイケースであるとは思う（笑）しかし何を基準にケースバイケースとするのかが重要だ。リファクタリングへの工数分配を考えるには大きく以下の2つの基準があるだろう。

1. 機能開発の緊急度
2. リファクタリング対象の悪性度

機能開発の緊急度とは、その開発がどれぐらいのスピードで完了しなければならないかである。リファクタリング対象の悪性度とは、リファクタリングをしないことによる損害の大きさである。

基本的にはこの2点をそれぞれ検討してリファクタリングの工数量を調整すると思う。機能開発の緊急度が高いほどリファクタリングをやらないし、リファクタリング対象の悪性度が高いほどリファクタリングに時間を使う。

尚、1と2の尺度を考えるときに気をつけたいのは、なるべく時間だけで考えるということである。1はあと何日で完了すべきなどで分かりやすいが、2についてもコードが美しさなどは直接の判断基準に入れない。その設計のまずさによって、レビュー時間がどれぐらい伸びるとか、どれぐらいバグが発生しそうで、その修正にどれぐらい時間がとられるか、などのように考えていく。

このように考えると、どんなに設計に問題があっても今後触れずに済むなら、悪性度は低くリファクタリングをする必要がないという判断もしやすい。

### リファクタリングに着手してから機能開発するか、機能開発をしてからリファクタリングに着手するか

多くの人は、リファクタリングに着手してから機能開発するような気がする。このメリットは明らかで、機能開発の対象物についてリファクタリングを先に済ませることで、機能開発を素早く実施できる。一方デメリットもあって、リファクタリングを先に着手すると機能開発に使える時間が圧迫されてしまう。圧迫されるということは、機能開発を完了できないリスクが高まるということである。

そこで、反対に機能開発を先に進めてリファクタリングをその後に、とすることで、機能開発を完了できる可能性を高められる。逆にリファクタリングに使える時間が減少する恐れがあるが、機能開発を完了できないよりは許されることが殆どだろう。

結論としては、基本的にまずリファクタリングをするのが良い。先に述べたリファクタリング先のメリットに加え、以下の理由があるからである。

- 実施したリファクタリングの有効性を速やかに評価できる
    - リファクタリングしたコードベースを機能開発ですぐに利用することでフィードバックがかかる
    - 次のリファクタリングのためのナレッジにできる
    - リファクタリングの上達速度が上がる
- リファクタリングは先行投資的な活動で習慣的に行うべき
    - 先に実施することで確実にリファクタリングに着手できるので習慣化しやすい
    - 後に実施すると、機能開発に圧迫されてリファクタリングがさぼりがちとなり、悪循環に陥る

ただし、リファクタリングを先にすることで、機能開発の完了にリスクが生じることには変わりない。このリスクを減らすためにはリファクタリングの量を少なめにしてから、機能開発に着手するのが良いだろう。そして機能開発が完了して余裕があればまたリファクタリングをすれば良い。
