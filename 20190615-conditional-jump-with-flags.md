---
labels: [Shell, Tips]
---

# シェルのフラグ分岐をコンパクトに書く

シェルスクリプトのTips。

外部から指定した環境変数でシェルの動作を分岐させたいことがある。このとき、以下のように条件文を書くと綺麗になる。

```
#!/bin/sh

if ($FLAG)
then
        echo 'FLAG is true'
else
        echo 'FLAG is false'
fi
```

実行してみる。
```
$ FLAG=true ./flag.sh
FLAG is true
$ FLAG=false ./flag.sh
FLAG is false
```

ポイントは、`FLAG`に設定する`true`, `false`を文字列としてではなく、コマンドとして解釈させる点である。

コマンドではなく文字列として`FLAG`を扱う場合、野暮ったい条件文になってしまう。

```
if [ "$FLAG" = true ]
then
        echo 'FLAG is true'
else
        echo 'FLAG is false'
fi
```
