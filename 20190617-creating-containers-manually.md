---
labels: [社内勉強会]
---

# chroot/unshareで手動でコンテナを作成する

コンテナ技術を支える仕組みであるchroot, unshareを紹介する。

## chroot

ルートディレクトリを指定して特定コマンドを実行する。

```
$ chroot NEWROOT COMMAND
```

本コマンドにより、実行されたコマンドには隔離されたディレクトリツリーのみが見える。コンテナはコンテナ間で互いにディレクトリが疎になっているように見えるが、それはchrootで実現している。

## unshare

コンテナ実現にあたっては、ディレクトリツリーを隔離しただけでは不十分である。unshareという機能を用いることで、ディレクトリツリー以外のリソースについてもプロセス間にて見えないようにできる。なおunshareはLinuxにおけるNamespace機能を実現するためのコマンド及びシステムコールである。

helpを見ると、どのようなNamespaceがあるかが分かる。

```
$ unshare --help

Usage:
 unshare [options] [<program> [<argument>...]]

Run a program with some namespaces unshared from the parent.

Options:
 -m, --mount[=<file>]      unshare mounts namespace
 -u, --uts[=<file>]        unshare UTS namespace (hostname etc)
 -i, --ipc[=<file>]        unshare System V IPC namespace
 -n, --net[=<file>]        unshare network namespace
 -p, --pid[=<file>]        unshare pid namespace
 -U, --user[=<file>]       unshare user namespace
 -C, --cgroup[=<file>]     unshare cgroup namespace
 :
 :
```

## デモ

### chrootでディレクトリツリーを隔離する

```
$ pwd
/home/ubuntu/pj/manual-containers
$ mkdir myroot && cd myroot
$ cp -r /bin /lib .
$ sudo chroot . /bin/bash
$ pwd
/
$ ls
bin  dev  lib  lib64  proc
```

ファイルシステムが隔離されていることが分かる。

### unshareでプロセス名空間を隔離する

```
$ pwd
/home/ubuntu/pj/manual-containers/myroot
$ sudo unshare -p chroot . bash

# psに必要なファイルシステムを準備
$ mkdir -p /proc
$ mount -t proc proc /proc

$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
0            1     0  0 00:07 ?        00:00:00 bash
0            4     1  0 00:07 ?        00:00:00 ps -ef
```

## 続き

今回chrootとunshareにより各種リソースの名前空間が隔離できることがわかった。しかし、まだ隔離すべきものとしてCPU/メモリ/ディスク等の物理リソース割り当てがある。これらの隔離のためにはcgroupsという機能が用いる。次回cgroupsについて紹介する。
