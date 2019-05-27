---
labels: [Tips, Linux]
---

# pgrepでプロセスの詳細情報を見る

プロセス検索方法のTips。

## ps -ef | grep xxx

Linuxでプロセス一覧を見るコマンドといえば`ps`である。特定の名前のプロセスを検索するには、以下のように`ps -ef | grep xxx`する。

```
# systemdプロセスを検索する
$ ps -ef | grep systemd
root         1     0  0 May25 ?        00:00:04 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
root      1929     1  0 May25 ?        00:00:02 /usr/lib/systemd/systemd-journald
root      1964     1  0 May25 ?        00:00:00 /usr/lib/systemd/systemd-udevd
dbus      2681     1  0 May25 ?        00:00:02 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
root      2696     1  0 May25 ?        00:00:01 /usr/lib/systemd/systemd-logind
ec2-user 30751 30710  0 15:54 pts/0    00:00:00 grep --color=auto systemd
```

問題は、上記にように`grep`プロセス自体も含まれてしまうことである。

## ps -ef | grep [x]xx

`grep`プロセスを結果から省くには、正規表現を駆使して以下のようにする。

```
$ ps -ef | grep [s]ystemd
root         1     0  0 May25 ?        00:00:04 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
root      1929     1  0 May25 ?        00:00:02 /usr/lib/systemd/systemd-journald
root      1964     1  0 May25 ?        00:00:00 /usr/lib/systemd/systemd-udevd
dbus      2681     1  0 May25 ?        00:00:02 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
root      2696     1  0 May25 ?        00:00:01 /usr/lib/systemd/systemd-logind
```

これで`grep`プロセスが見えなくなる。

## pgrep -a xxx

上記の正規表現を使う方法はややトリッキーな感じがする。タイプもしにくい。

より直感的に使えるコマンドが`pgrep`である。`-a`オプションで起動時のコマンドラインを全表示できる。

```
$ pgrep -a systemd
1 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
1929 /usr/lib/systemd/systemd-journald
1964 /usr/lib/systemd/systemd-udevd
2696 /usr/lib/systemd/systemd-logind
```

しかし、`pgrep`の問題は`ps -f`相当の詳細表示ができないことだ。例えば、起動ユーザーや起動時刻が分からない。

## pgrep xxx | xargs ps -f

こちらが最近よく使っているコマンドである。`pgrep`でプロセスIDを取得してから、`xargs ps -f`で詳細表示をする。
```
$ pgrep systemd | xargs ps -f
UID        PID  PPID  C STIME TTY      STAT   TIME CMD
root         1     0  0 May25 ?        Ss     0:04 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
root      1929     1  0 May25 ?        Ss     0:02 /usr/lib/systemd/systemd-journald
root      1964     1  0 May25 ?        Ss     0:00 /usr/lib/systemd/systemd-udevd
root      2696     1  0 May25 ?        Ss     0:01 /usr/lib/systemd/systemd-logind
```

自然なコマンドラインで、かつ得られる情報も完璧である。`ps -ef | grep xxx`と異なり`ps`のヘッダーが出る点も優れる。

## 参考

- [linux - How to get pgrep to display full process info - Server Fault](https://serverfault.com/questions/77162/how-to-get-pgrep-to-display-full-process-info)
