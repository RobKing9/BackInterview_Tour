- [ ] ## 1、defer与结构体

### 下面代码段输出什么？

```golang
type Person struct {
    age int
}

func main() {
    person := &Person{28}

    // 1. 
    defer fmt.Println(person.age)

    // 2.
    defer func(p *Person) {
        fmt.Println(p.age)
    }(person)  

    // 3.
    defer func() {
        fmt.Println(person.age)
    }()

    person.age = 29
}
```

### 答案及解析

```golang
参考答案及解析：29 29 28。变量 person 是一个指针变量 。

1.person.age 此时是将 28 当做 defer 函数的参数，会把 28 缓存在栈中，等到最后执行该 defer 语句的时候取出，即输出 28；

2.defer 缓存的是结构体 Person{28} 的地址，最终 Person{28} 的 age 被重新赋值为 29，所以 defer 语句最后执行的时候，依靠缓存的地址取出的 age 便是 29，即输出 29；

3.闭包引用，输出 29；

又由于 defer 的执行顺序为先进后出，即 3 2 1，所以输出 29 29 28。
```

- [ ] ## 2、defer与结构体

### 下面代码段输出什么？

```golang
type Person struct {
    age int
}

func main() {
    person := &Person{28}

    // 1.
    defer fmt.Println(person.age)

    // 2.
    defer func(p *Person) {
        fmt.Println(p.age)
    }(person)

    // 3.
    defer func() {
        fmt.Println(person.age)
    }()

    person = &Person{29}
}
```

### 答案及解析

```golang
参考答案及解析：29 28 28。这道题在第 19 天题目的基础上做了一点点小改动，前一题最后一行代码 person.age = 29 是修改引用对象的成员 age，这题最后一行代码 person = &Person{29} 是修改引用对象本身，来看看有什么区别。

1处.person.age 这一行代码跟之前含义是一样的，此时是将 28 当做 defer 函数的参数，会把 28 缓存在栈中，等到最后执行该 defer 语句的时候取出，即输出 28；

2处.defer 缓存的是结构体 Person{28} 的地址，这个地址指向的结构体没有被改变，最后 defer 语句后面的函数执行的时候取出仍是 28；

3处.闭包引用，person 的值已经被改变，指向结构体 Person{29}，所以输出 29.

由于 defer 的执行顺序为先进后出，即 3 2 1，所以输出 29 28 28。
```

## 3、空接口

### A、B、C、D 哪些选项有语法错误？

```golang
type S struct {
}

func f(x interface{}) {
}

func g(x *interface{}) {
}

func main() {
    s := S{}
    p := &s
    f(s) //A
    g(s) //B
    f(p) //C
    g(p) //D
}
```

### 答案及解析

```golang
参考答案及解析：BD。函数参数为 interface{} 时可以接收任何类型的参数，包括用户自定义类型等，即使是接收指针类型也用 interface{}，而不是使用 *interface{}。
```

## 4、return 之后的 defer 语句

### 下面这段代码输出什么？

```golang
var a bool = true
func main() {
    defer func(){
        fmt.Println("1")
    }()
    if a == true {
        fmt.Println("2")
        return
    }
    defer func(){
        fmt.Println("3")
    }()
}
```

### 答案及解析

```golang
参考答案及解析：2 1。defer 关键字后面的函数或者方法想要执行必须先注册，return 之后的 defer 是不能注册的， 也就不能执行后面的函数或方法。
```

## 5、defer与子函数

### 下面这段代码输出什么？

```golang
func main() {
    a := 1
    b := 2
    defer calc("1", a, calc("10", a, b))
    a = 0
    defer calc("2", a, calc("20", a, b))
    b = 1
}

func calc(index string, a, b int) int {
    ret := a + b
    fmt.Println(index, a, b, ret)
    return ret
}
```



### 答案及解析

```golang
参考答案及解析：
10 1 2 3
20 0 2 2
2 0 2 2
1 1 3 4
程序执行到 main() 函数三行代码的时候，会先执行 calc() 函数的 b 参数，即：calc("10",a,b)，输出：10 1 2 3，得到值 3，因为
defer 定义的函数是延迟函数，故 calc("1",1,3) 会被延迟执行；
程序执行到第五行的时候，同样先执行 calc("20",a,b) 输出：20 0 2 2 得到值 2，同样将 calc("2",0,2) 延迟执行；
程序执行到末尾的时候，按照栈先进后出的方式依次执行：calc("2",0,2)，calc("1",1,3)，则就依次输出：2 0 2 2，1 1 3 4。
```

## 6、变量作用域与指针

### 下面这段代码输出什么？如果编译错误的话，为什么？

```golang
var p *int

func foo() (*int, error) {
    var i int = 5
    return &i, nil
}

func bar() {
    //use p
    fmt.Println(*p)
}

func main() {
    p, err := foo()  //output: 0x14dc80, 0×0, *int
    if err != nil {
        fmt.Println(err)
        return
    }
    bar()  //output: 0x2081c6020, 0x20818a258, *int
    fmt.Println(*p)
}
```

- A. 5 5
- B. runtime error

### 答案及解析

```golang
参考答案及解析：B。知识点：变量作用域。问题出在操作符:=，对于使用:=定义的变量，如果新变量与同名已定义的变量不在同一个作用域中，那么 Go 会新定义这个变量。对于本例来说，main() 函数里的 p 是新定义的变量，会遮住全局变量 p，导致执行到bar()时程序，全局变量 p 依然还是 nil，程序随即 Crash。

正确的做法是将 main() 函数修改为：

func main() {
    var err error
    p, err = foo()
    if err != nil {
        fmt.Println(err)
        return
    }
    bar()
    fmt.Println(*p)
}
```

## 7、切片，数组比较

### 下面的代码有什么问题？

```golang
func main() {
    fmt.Println([...]int{1} == [2]int{1})
    fmt.Println([]int{1} == []int{1})
}
```



### 答案及解析

```golang
参考答案及解析：有两处错误

go 中不同类型是不能比较的，而数组长度是数组类型的一部分，所以 […]int{1} 和 [2]int{1} 是两种不同的类型，不能比较；

切片是不能比较的；数组是可以比较的，必须长度和值都一样返回true；如果长度一样，值不同返回false
```

## 8、切片遍历 循环

### 下面这段代码能否正常结束？如果可以？输出什么？

```golang
func main() {
    v := []int{1, 2, 3}
    for i := range v {
        v = append(v, i)
    }
    fmt.Println(v)
}
```



### 答案及解析

```golang
参考答案及解析：不会出现死循环，能正常结束。
循环次数在循环开始前就已经确定，循环内改变切片的长度，不影响循环次数。
```

## 9、切片遍历与goroutine

### 下面这段代码输出什么？为什么？

```golang
func main() {

    var m = [...]int{1, 2, 3}

    for i, v := range m {
        go func() {
            fmt.Println(i, v)
        }()
    }

    time.Sleep(time.Second * 3)
}
```



### 答案及解析

```golang
参考答案及解析：
2 3
2 3
2 3
for range 使用短变量声明(:=)的形式迭代变量，需要注意的是，变量 i、v 在每次循环体中都会被重用，而不是重新声明。

各个 goroutine 中输出的 i、v 值都是 for range 循环结束后的 i、v 最终值，而不是各个goroutine启动时的i, v值。可以理解为闭包引用，使用的是上下文环境的值。

两种可行的 fix 方法:

1.使用函数传递

for i, v := range m {
    go func(i,v int) {
        fmt.Println(i, v)
    }(i,v)
}
2.使用临时变量保留当前值

for i, v := range m {
    i := i           // 这里的 := 会重新声明变量，而不是重用
    v := v
    go func() {
        fmt.Println(i, v)
    }()
}
```




## 10、defer与函数参数，recover

### 下面这段代码输出什么

```golang
func f(n int) (r int) {
    defer func() {
        r += n
        recover()
    }()

    var f func()

    defer f()
    f = func() {
        r += 2
    }
    return n + 1
}

func main() {
    fmt.Println(f(3))
}
```



###  答案及解析

```golang
参考答案及解析：7。第一步执行r = n +1，接着执行第二个 defer，由于此时 f() 未定义，引发异常，随即执行第一个 defer，异常被 recover()，程序正常执行，最后 return。
```

## 11、数组遍历

### 下面这段代码输出什么？

```golang
func main() {
    var a = [5]int{1, 2, 3, 4, 5}
    var r [5]int

    for i, v := range a {
        if i == 0 {
            a[1] = 12
            a[2] = 13
        }
        r[i] = v
    }
    fmt.Println("r = ", r)
    fmt.Println("a = ", a)
}
```



### 答案及解析

```golang
参考答案及解析：

r =  [1 2 3 4 5]
a =  [1 12 13 4 5]
range 表达式是副本参与循环，就是说例子中参与循环的是 a 的副本，而不是真正的 a。就这个例子来说，假设 b 是 a 的副本，则 range 循环代码是这样的：

for i, v := range b {
    if i == 0 {
        a[1] = 12
        a[2] = 13
    }
    r[i] = v
}
因此无论 a 被如何修改，其副本 b 依旧保持原值，并且参与循环的是 b，因此 v 从 b 中取出的仍旧是 a 的原值，而非修改后的值。

如果想要 r 和 a 一样输出，修复办法：

func main() {
    var a = [5]int{1, 2, 3, 4, 5}
    var r [5]int

    for i, v := range &a {
        if i == 0 {
            a[1] = 12
            a[2] = 13
        }
        r[i] = v
    }
    fmt.Println("r = ", r)
    fmt.Println("a = ", a)
}
输出：

r =  [1 12 13 4 5]
a =  [1 12 13 4 5]
修复代码中，使用 *[5]int 作为 range 表达式，其副本依旧是一个指向原数组 a 的指针，因此后续所有循环中均是 &a 指向的原数组亲自参与的，因此 v 能从 &a 指向的原数组中取出 a 修改后的值。
```

## 12、可变函数，append

### 下面这段代码输出什么？

```golang
func change(s ...int) {
    s = append(s,3)
}

func main() {
    slice := make([]int,5,5)
    slice[0] = 1
    slice[1] = 2
    change(slice...)
    fmt.Println(slice)
    change(slice[0:2]...)
    fmt.Println(slice)
}
```

### 答案及解析

```golang
参考答案及解析：

[1 2 0 0 0]
[1 2 3 0 0]
知识点：可变函数、append()操作。Go 提供的语法糖...，可以将 slice 传进可变函数，不会创建新的切片。第一次调用 change() 时，append() 操作使切片底层数组发生了扩容，原 slice 的底层数组不会改变；第二次调用change() 函数时，使用了操作符[i,j]获得一个新的切片，假定为 slice1，它的底层数组和原切片底层数组是重合的，不过 slice1 的长度、容量分别是 2、5，所以在 change() 函数中对 slice1 底层数组的修改会影响到原切片。
```

## 13、切片遍历

### 下面这段代码输出什么？

```golang
func main() {
    var a = []int{1, 2, 3, 4, 5}
    var r [5]int

    for i, v := range a {
        if i == 0 {
            a[1] = 12
            a[2] = 13
        }
        r[i] = v
    }
    fmt.Println("r = ", r)
    fmt.Println("a = ", a)
}
```



### 答案及解析

```golang
参考答案及解析：

r =  [1 12 13 4 5]
a =  [1 12 13 4 5]
这道题是 11题 的一个解决办法，这的 a 是一个切片，那切片是怎么实现的呢？切片在 go 的内部结构有一个指向底层数组的指针，当 range 表达式发生复制时，副本的指针依旧指向原底层数组，所以对切片的修改都会反应到底层数组上，所以通过 v 可以获得修改后的数组元素。
```

## 14、切片遍历

### 下面这段代码输出结果正确吗？

```golang
type Foo struct {
    bar string
}
func main() {
    s1 := []Foo{
        {"A"},
        {"B"},
        {"C"},
    }
    s2 := make([]*Foo, len(s1))
    for i, value := range s1 {
        s2[i] = &value
    }
    fmt.Println(s1[0], s1[1], s1[2])
    fmt.Println(s2[0], s2[1], s2[2])
}
输出：
{A} {B} {C}
&{A} &{B} &{C}
```



### 答案及解析

```golang
参考答案及解析：s2 的输出结果错误。s2 的输出是 &{C} &{C} &{C}，在 第 30 天 的答案解析第二题，我们提到过，for range 使用短变量声明(:=)的形式迭代变量时，变量 i、value 在每次循环体中都会被重用，而不是重新声明。所以 s2 每次填充的都是临时变量 value 的地址，而在最后一次循环中，value 被赋值为{c}。因此，s2 输出的时候显示出了三个 &{c}。

可行的解决办法如下：

for i := range s1 {
    s2[i] = &s1[i]
}xxxxxxxxxx 参考答案参考答案及解析：s2 的输出结果错误。s2 的输出是 &{C} &{C} &{C}，在 第 30 天 的答案解析第二题，我们提到过，for range 使用短变量声明(:=)的形式迭代变量时，变量 i、value 在每次循环体中都会被重用，而不是重新声明。所以 s2 每次填充的都是临时变量 value 的地址，而在最后一次循环中，value 被赋值为{c}。因此，s2 输出的时候显示出了三个 &{c}。可行的解决办法如下：for i := range s1 {    s2[i] = &s1[i]}
```

## 15、多重赋值

### 下面代码输出正确的是？

```golang
func main() {
    i := 1
    s := []string{"A", "B", "C"}
    i, s[i-1] = 2, "Z"
    fmt.Printf("s: %v \n", s)
}
A. s: [Z,B,C]

B. s: [A,Z,C]
```



### 答案及解析

```golang
参考答案及解析：A。知识点：多重赋值。

多重赋值分为两个步骤，有先后顺序：

计算等号左边的索引表达式和取址表达式，接着计算等号右边的表达式；

赋值；

所以本例，会先计算 s[i-1]，等号右边是两个表达式是常量，所以赋值运算等同于 i, s[0] = 2, "Z"。
```

## 16、switch语句

### 关于switch语句，下面说法正确的有?

```golang
A. 条件表达式必须为常量或者整数；

B. 单个case中，可以出现多个结果选项；

C. 需要用break来明确退出一个case；

D. 只有在case中明确添加fallthrough关键字，才会继续执行紧跟的下一个case；题目
```



### 答案及解析

```golang
参考答案及解析：BD
```

## 17、类型断言

### 如果 Add() 函数的调用代码为：

```golang
func main() {
    var a Integer = 1
    var b Integer = 2
    var i interface{} = &a
    sum := i.(*Integer).Add(b)
    fmt.Println(sum)
}
则Add函数定义正确的是()

A.
type Integer int
func (a Integer) Add(b Integer) Integer {
        return a + b
}

B.
type Integer int
func (a Integer) Add(b *Integer) Integer {
        return a + *b
}

C.
type Integer int
func (a *Integer) Add(b Integer) Integer {
        return *a + b
}

D.
type Integer int
func (a *Integer) Add(b *Integer) Integer {
        return *a + *b
}
```



### 答案及解析

```golang
参考答案及解析：AC。知识点：类型断言、方法集
```

[接口讲解](https://mp.weixin.qq.com/s?__biz=MzI2MDA1MTcxMg==&mid=2648466700&idx=1&sn=25c48d78dcfad6c70330cd36dd749e53&chksm=f2474363c530ca75132454e4e10e40659310a073e2f9d30ad9697d4abf7c2b9e5aa9adee58bf&scene=21#wechat_redirect)

## 18、bool变量

### 关于 bool 变量 b 的赋值，下面错误的用法是？

```golang
A. b = true

B. b = 1

C. b = bool(1)

D. b = (1 == 2)
```

### 答案及解析

```golang
参考答案及解析：BC。
```

## 19、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

## 20、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

## 21、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

## 22、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

## 23、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

## 24、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

