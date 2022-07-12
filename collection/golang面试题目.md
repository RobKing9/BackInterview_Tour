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

## 6、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

## 7、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

## 8、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

## 9、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

## 10、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

## 11、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

## 12、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

## 13、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

## 14、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

## 15、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

## 16、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

## 17、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
```

## 18、

### 题目

```golang
题目
```



### 答案及解析

```golang
参考答案
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

