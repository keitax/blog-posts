---
labels: ["学習メモ", "Algorithms"]
---

# Algorithms 1.1-1.3

Robert Sedgwickの[Algorithms, 4th Edition](https://algs4.cs.princeton.edu/home/)を学習していく。

## 1 Fundamentals

### 1.1 Programming Model

- Javaの基本的な構文や概念の話
- 今回はGoやるのでスキップ

### 1.2 Data Abstraction

- OOP, 抽象データ型の話
- データ型: 値の組とその操作の組
        - 操作まで含める？
- 抽象データ型（ADT）: データ型の利用者がその内部表現を知らなくてよいデータ型
- オブジェクト: データ型の値としても扱える、3点で特徴付けられるもの
  - 状態
  - 識別性
  - 振る舞い
- OOPの機能をつかえばADTの利用者と内部実装の開発を分離しやすい
- カプセル化でモジュラープログラミングをやりやすくする 
  - モジュラープログラミング: ソフトウェア要素を個別に開発してから組み合わせていく手法
- ADTはアルゴリズム学習によい
  - アルゴリズムが達成しなければならないことと、アルゴリズムの利用方法を分けて考えるためのフレームワークを提供してくれる

### 1.3 Bags, Queues, and Stacks

- Bag: remove()をサポートしないコレクション型
- Queue, Stack周りはスキップ

#### Exercises

##### 1.

Stackの`IsFill()`を実装する。

```
package main

import "fmt"

type FixedCapacityStack struct {
	a []interface{}
	n int
}

func New(capacity int) *FixedCapacityStack {
	return &FixedCapacityStack{
		a: make([]interface{}, capacity),
		n: 0,
	}
}

func (fcs *FixedCapacityStack) Push(v interface{}) {
	fcs.a[fcs.n] = v
	fcs.n++
}

func (fcs *FixedCapacityStack) Pop() interface{} {
	fcs.n--
	return fcs.a[fcs.n]
}

func (fcs *FixedCapacityStack) IsFill() bool {
	return len(fcs.a) <= fcs.n
}

func main() {
	s := New(2)
	fmt.Println(s.IsFill())
	s.Push(0)
	fmt.Println(s.IsFill())
	s.Push(0)
	fmt.Println(s.IsFill())
}
```

##### 2.

Stackの使い方の問題。GoなのでSliceで代用。

```
package main

import (
	"fmt"
)

func main() {
	s := []string{}
	var v string

	s = append(s, "it")
	s = append(s, "was")

	s, v = s[:len(s)-1], s[len(s)-1]
	fmt.Println(v)

	s = append(s, "the")
	s = append(s, "best")

	s, v = s[:len(s)-1], s[len(s)-1]
	fmt.Println(v)

	s = append(s, "of")
	s = append(s, "times")

	s, v = s[:len(s)-1], s[len(s)-1]
	fmt.Println(v)
	s, v = s[:len(s)-1], s[len(s)-1]
	fmt.Println(v)
	s, v = s[:len(s)-1], s[len(s)-1]
	fmt.Println(v)

	s = append(s, "it")
	s = append(s, "was")

	s, v = s[:len(s)-1], s[len(s)-1]
	fmt.Println(v)

	s = append(s, "the")

	s, v = s[:len(s)-1], s[len(s)-1]
	fmt.Println(v)
	s, v = s[:len(s)-1], s[len(s)-1]
	fmt.Println(v)

	fmt.Printf("%v\n", s)
}
```

##### 3.

Q. 0~9のorderedなpush()とpop()を混ぜて実行したときにありえないのは？<br>
A. 以下の3つ。数値が増加するとき必ず新しくpush()した値になる法則を満たしてない

```
(b)  4 6 8 7 5 3 2 9 0 1
(f)  0 4 6 5 3 8 1 7 2 9
(g)  1 4 7 9 8 6 5 3 0 2
```

##### 4. 

括弧が揃っているかをStackで検証する。

```
package main

import (
	"fmt"
)

func IsBalanced(in string) bool {
	sym := map[rune]rune{
		')': '(',
		'}': '{',
		']': '[',
	}
	var s []rune
	for _, c := range in {
		switch c {
		case '[', '{', '(':
			s = append(s, c)
		case ')', '}', ']':
			if len(s) <= 0 {
				return false
			}
			var v rune
			s, v = s[:len(s)-1], s[len(s)-1]
			if v != sym[c] {
				return false
			}
		}
	}
	return len(s) <= 0
}

func main() {
	var in string
	fmt.Scanf("%s\n", &in)
	if IsBalanced(in) {
		fmt.Println("balanced")
	} else {
		fmt.Println("unbalanced")
	}
}
```