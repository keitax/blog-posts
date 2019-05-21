---
labels: [プログラミング, 社内勉強会, Go]
---

# tail -fを実装する

`tail -f` とはエンジニアであれば誰もが使ったことのある、おなじみのコマンドである。対象ファイルへの出力をリアルタイムに表示することから、稼働中システムのログを見る際に重宝する。

最近この `tail -f` に相当する動作、つまりログファイルをリアルタイムに読み込んでいく要件のアプリケーションを実装する機会があった。
そのときに得た知見を一般化し、本記事としたい。

## tail -fとtail

`tail -f` はそのオプションなし版である `tail` と動作が大きく異なる。`tail` は単にファイル末尾部分を読んだ後に終了するだけだが、`tail -f` は、コマンドが終了せず対象ファイルへの出力を待ち続ける。そして新しくファイルへ出力された内容をすぐに標準出力へ出力する。

この `tail -f` の動作は一見単純そうに見えるが、その実装はやや複雑である。どのような実装となるのか見ていこう。

尚、Goにより `tail -f` を実装し、その実装例によって `tail -f` 実装方法の説明とする。

## tail -fの大まかな流れ

`tail -f` の実装を大まかに書くと以下のようになる。

1. ファイルを開く
2. ファイルポインタをファイル末尾とする 
3. ファイル書き込みイベントを捕捉する
4. ファイル増加分を読む
5. 3, 4を繰り返す

上記のうち特に重要なのは「3. ファイル書き込みイベントを補足する」である。

## ファイル書き込みイベントの捕捉

ここでの、ファイル書き込みイベントとは、ファイルへ何らかのデータが書き込まれたタイミングのことである。どのように捉えられるだろうか。以下2つの手法がある。

1. ファイル情報をポーリングする
2. ファイルシステムイベントを監視する

どちらの手法でも `tail -f` を実装できるが、前者の方がより単純である一方、後者の方がシステムコール数が抑えられ、パフォーマンス的にやや有利という違いがある。

それぞれの手法の詳細と実装例を示す。

### ファイル情報をポーリングする

こちらの実装の方が簡単である。LinuxのようなPOSIX準拠のOSでは、システムコール `stat(2)` を利用できる。`stat(2)` は主にファイルサイズなどのファイルの詳細情報を取得する。

ファイル容量が増加したときがファイル書き込みされたときである。よって `stat(2)` で得られるファイルサイズのよりファイル書き込みされたかどうかを判定できる。

具体的な実装は以下のようになるだろう。

	// tail_polling.go
	
	package main
	
	import (
		"bufio"
		"fmt"
		"io"
		"os"
		"time"
	)
	
	func main() {
		// 対象ファイルの決定
		targetFile := os.Args[1]
	
		f, err := os.Open(targetFile)
		if err != nil {
			panic(err)
		}
	
		// ファイルポインタをファイル末尾に移動
		st, err := os.Stat(targetFile)
		if err != nil {
			panic(err)
		}
		f.Seek(st.Size(), 0)
	
		r := bufio.NewReader(f)
		buf := make([]byte, 32)
	
		lastFileSize := int64(0)
	
		// 500 msごとにファイルを監視・読み込み
		for range time.Tick(500 * time.Millisecond) {
			st, err := os.Stat(targetFile)
			if err != nil {
				panic(err)
			}
	
			// ファイルサイズが増加しているかどうか
			if lastFileSize < st.Size() {
				for {
					_, err := r.Read(buf)
					if err == io.EOF {
						break
					}
					if err != nil {
						panic(err)
					}
					fmt.Print(string(buf))
				}
				lastFileSize = st.Size()
			}
		}
	}

### ファイルシステムイベントを捕捉する

ポーリングではなく、ファイルシステムイベントを捕捉してファイル容量増加を検知する手法もある。ポーリングによる実装は、上記の実装では500 ms単位としているように厳密なリアルタイム処理とできない。また500 ms毎に `stat(2)` を呼びだすことから非効率な欠点がある。ファイルシステムイベントを捕捉する手法ではこれらの欠点を解消できる。

尚、ファイルシステムイベントとは具体的にはファイルシステムに関する以下のようなイベントである。

- ファイルオープン/クローズ
- ファイル読み込み・書き込み
- ファイル移動

詳細はinotifyの[manページ][1]を見るのが良い。

尚、Goにてinotifyを直接扱うのは骨が折れる。Goでファイルシステムイベントを捕捉するには、inotifyをラップした [fsnotify][2]を用いる。fsnotifyは、inotify以外のファイルシステムイベントAPIにも対応しており、プログラミングが簡単になるだけでなくLinux以外のOSにも対応できるようになる。

fsnotifyによる`tail -f`の実装例を示す。

	// tail_fsnotify.go
	
	package main
	
	import (
		"bufio"
		"fmt"
		"github.com/fsnotify/fsnotify"
		"io"
		"os"
	)
	
	func main() {
		// 対象ファイルの決定
		targetFile := os.Args[1]
	
		f, err := os.Open(targetFile)
		if err != nil {
			panic(err)
		}
		defer f.Close()
	
		// ファイルポインタをファイル末尾に移動
		st, err := os.Stat(targetFile)
		if err != nil {
			panic(err)
		}
		f.Seek(st.Size(), 0)
	
		r := bufio.NewReader(f)
		buf := make([]byte, 32)
	
		watcher, err := fsnotify.NewWatcher()
		if err != nil {
			panic(err)
		}
		defer watcher.Close()
	
		// 監視対象のファイルを設定する
		watcher.Add(targetFile)
	
		// ファイルシステムイベントをchannelで受ける
		for event := range watcher.Events {
			if event.Op == fsnotify.Write {
				for {
					_, err := r.Read(buf)
					if err == io.EOF {
						break
					}
					if err != nil {
						panic(err)
					}
					fmt.Print(string(buf))
				}
			}
		}
	
	}

## 発展的な話題

ここまで、`tail -f` のGoによる実装方法を述べた。`tail -f` のようなログファイル読み込み処理を本番アプリケーションに入れる場合には、以下のような、より発展的な事項を検討して実装する必要がある。

- 対象ファイルがローテートされてもファイルを読み続けられるか
	- `tail -F` 相当の処理を実装する必要がある
- アプリケーション再起動時の読み込み位置はファイル末尾で良いか
	- ファイル末尾では、アプリケーション停止時に増加したログを読み込めない

特に、ログ転送ミドルウェアの[fluentd tailプラグイン][3]などは、上記を考慮した堅牢な実装となっている。

[1]:	http://man7.org/linux/man-pages/man7/inotify.7.html
[2]:	https://github.com/fsnotify/fsnotify
[3]:	https://docs.fluentd.org/input/tail
