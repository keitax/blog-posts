---
labels: [旅行, ブログ実装]
---

# 金沢旅行1日目—開発合宿@金沢

ゴールデンウィークということで金沢に来た。長期休暇にリフレッシュに小旅行に行くことが多い。

とはいえ主目的は観光ではなくソフトウェア開発である（笑）

旅行とソフトウェア開発は相性の良い活動だと思う。
普段と異なる場所で作業することで、精神的に煮詰まりにくいような気がするからである。
リフレッシュにもなるし進捗も出るし最高なのだ。

## ブログ実装

何を開発するかというとブログサービスである。

現在、この記事を書いているが、記事を表示するブログサービスはまだないという状況だ。
今回の金沢旅では、ブログサービスのMVP（Minimum Viable Product）を完成させ、この記事を表示することを目標としたい。

はてなブログやBloggerなど、世には優れたブログサービスが存在しているが、あえて自作としたい。

私にはブログ投稿のUXが重要で、普段からIntelliJでMarkdownメモをとる習慣がある。同じようなUXでブログに記事投稿できると嬉しい。
[Jekyll](https://jekyllrb.com)などMarkdownベースのブログジェネレータも試したが細かいところで自分好みでない部分があった。

またブログサービスは、個人でのプロダクト制作に格好のテーマで、簡単に実装できる割にドッグフーディングしやすく、技術を試すためのプラットフォームに適する。
例えば今回はAWSにブログを載せる。そうすると必然的にAWSのエコシステムを学ぶことになる。

## どんなブログにするか

私はエンジニアなので、ソフトウェアエンジニアリングのことしか書けないだろう。

対象読者としては、不特定多数の方というよりも、まずは同僚など私のことを知っている人に読んでもらえるような記事を書ければと思っている。

それでは、よろしくお願いします。
