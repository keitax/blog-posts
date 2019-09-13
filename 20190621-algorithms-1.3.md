---
labels: ["学習メモ", "Algorithms"]
---

# Algorithms 1.3

## 1.3 Exercises

### 10.

InfixToPostFix()の実装

```
func InfixToPostFix(expression string) string {
	isyms := strings.Split(expression, " ")
	var stack []string
	var psyms []string
	for _, sym := range isyms {
		switch sym {
		case "+", "-", "*", "/":
			stack = append(stack, sym)
		case "(":
		case ")":
			var v string
			stack, v = stack[:len(stack)-1], stack[len(stack)-1]
			psyms = append(psyms, fmt.Sprintf("%v", v))
		default:
			psyms = append(psyms, sym)
		}
	}
	return strings.Join(psyms, " ")
}
```

### 11.

EvaluatePostFix()の実装

```
func EvaluatePostfix(expression string) int {
	symbols := strings.Split(expression, " ")
	var stack []interface{}
	var l, r interface{}
	for _, sym := range symbols {
		switch sym {
		case "+":
			stack, r = stack[:len(stack)-1], stack[len(stack)-1]
			stack, l = stack[:len(stack)-1], stack[len(stack)-1]
			stack = append(stack, l.(int)+r.(int))
		case "-":
			stack, r = stack[:len(stack)-1], stack[len(stack)-1]
			stack, l = stack[:len(stack)-1], stack[len(stack)-1]
			stack = append(stack, l.(int)-r.(int))
		case "*":
			stack, r = stack[:len(stack)-1], stack[len(stack)-1]
			stack, l = stack[:len(stack)-1], stack[len(stack)-1]
			stack = append(stack, l.(int)*r.(int))
		case "/":
			stack, r = stack[:len(stack)-1], stack[len(stack)-1]
			stack, l = stack[:len(stack)-1], stack[len(stack)-1]
			stack = append(stack, l.(int)/r.(int))
		default:
			n, err := strconv.Atoi(sym)
			if err != nil {
				panic(err)
			}
			stack = append(stack, n)
		}
	}
	return stack[len(stack)-1].(int)
}
```

### 13.

Q. 0-9を順番にEnqueue()とDequeue()混ぜて繰り返したときにありえない出力は？

```
(a)  0 1 2 3 4 5 6 7 8 9
(b)  4 6 8 7 5 3 2 9 0 1 
(c)  2 5 6 7 4 8 9 3 1 0
(d)  4 3 2 1 0 5 6 7 8 9
```

A. b, c, d
キューに昇順にEnqueue()しているので、必ずDequeue()の結果も昇順になる。

### 14.

ResizingArrayQueueの実装。いわゆるリングバッファの実装。

```
type ResizingArrayQueue struct {
	item  []string
	first int
	last  int
	n     int
}

func NewResizingArrayQueue() *ResizingArrayQueue {
	return &ResizingArrayQueue{
		item:  make([]string, 1),
		first: 0,
		last:  0,
	}
}

func (q *ResizingArrayQueue) Enqueue(v string) {
	if q.n >= len(q.item) {
		q.Resize(len(q.item) * 2)
	}
	q.item[q.last] = v
	q.last = (q.last + 1) % len(q.item)
	q.n++
}

func (q *ResizingArrayQueue) Dequeue() string {
	v := q.item[q.first]
	q.first = (q.first + 1) % len(q.item)
	q.n--
	if q.n <= len(q.item)/4 {
		q.Resize(len(q.item) / 2)
	}
	return v
}

func (q *ResizingArrayQueue) Resize(size int) {
	new := make([]string, size)
	for i := 0; i < q.n; i++ {
		new[i] = q.item[(i+q.first)%len(q.item)]
	}
	q.item = new
	q.first = 0
	q.last = q.n
}
```
