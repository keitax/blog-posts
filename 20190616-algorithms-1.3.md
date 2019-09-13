---
labels: ["学習メモ", "Algorithms"]
---

# Algorithms 1.3

## 1.3 Exercises

### 5.

Q. n=50のとき、以下コードの出力結果は？
```
Stack<Integer> s = new Stack<Integer>();
while (n > 0) {
   s.push(n % 2);
   n = n / 2;
}
while (!s.isEmpty())
    System.out.print(s.pop());
System.out.println();
```

A. 50の2進数表現

- n%2で2進数表現での最小の桁をStackに記録する
- n/=2で最小の桁を削る
- このテクニックは一般的で2進数以外にも使える

10進数表現をsliceでとる場合
```
package main

import (
	"fmt"
)

func main() {
	n := 12345
	var st []int
	for n > 0 {
		st = append(st, n%10)
		n /= 10
	}
	fmt.Println(st)
}
```

### 6.

Q. 以下は何をする実装？
```
Stack<String> s = new Stack<String>();
while(!q.isEmpty())
   s.push(q.dequeue());
while(!s.isEmpty())
   q.enqueue(s.pop());
```
A. キューの中身をreverseする

### 7.

- peek()の実装

```
package main

import (
	"fmt"
)

type Stack struct {
	n     int
	items []interface{}
}

func New() *Stack {
	return &Stack{
		n:     0,
		items: make([]interface{}, 64),
	}
}
func (s *Stack) Push(item interface{}) {
	s.items[s.n] = item
	s.n++
}
func (s *Stack) Peek() interface{} {
	return s.items[s.n-1]
}

func main() {
	s := New()
	s.Push("a")
	fmt.Println(s.Peek())
	s.Push("b")
	fmt.Println(s.Peek())
} 
```