> * 原文地址：[Part 33: First Class Functions
](https://golangbot.com/first-class-functions/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

## 什么是头等函数

支持头等函数的语言允许将函数赋值给变量，作为参数传递给其他函数并从其他函数返回。 Go 支持头等函数功能。

在本教程中，我们将讨论头等函数的语法和各种用例。

## 匿名函数

让我们从一个简单的例子开始，它将一个函数赋给变量。
```golang
package main

import (  
    "fmt"
)

func main() {  
    a := func() {
        fmt.Println("hello world first class function")
    }
    a()
    fmt.Printf("%T", a)
}
```
[Run in playground](https://play.golang.org/p/Xm_ihamhlEv)

在上面的程序中的第 8 行，我们为变量`a`分配了一个函数。 这是将函数赋值给变量的语法。如果你仔细注意，分配给`a`的函数没有名称。这些没有名称的函数称为匿名函数。

调用此函数的唯一方法是使用变量`a`。我们使用 `a()`调用该函数，将打印`hello world first class function`。在第 12 行，我们打印变量`a`的类型，将打印`func()`。

运行该程序，将输出，
```plain
hello world first class function  
func()  
```

也可以不赋值给变量来调用匿名函数，让我们看看以下示例中怎么完成的此操作。

```golang
package main

import (  
    "fmt"
)

func main() {  
    func() {
        fmt.Println("hello world first class function")
    }()
}
```
[Run in playground](https://play.golang.org/p/Xm_ihamhlEv)

在上面的程序中，我们在第 8 行定义了一个匿名函数。 在函数定义之后，我们在行号中使用`()`调用函数。 该程序将输出，
```plain
hello world first class function  
```

也可以像任何其他函数一样将参数传递给匿名函数。

```golang
package main

import (  
    "fmt"
)

func main() {  
    func(n string) {
        fmt.Println("Welcome", n)
    }("Gophers")
}
```
[Run in playground](https://play.golang.org/p/9ttJ5Wi4fj4)

在上面的程序中的第 10 行，将字符串参数传递给匿名函数。运行此程序将打印，
```plain
Welcome Gophers  
```

## 用户定义的函数类型

就像定义自己的结构类型一样，我们也可以定义自己的函数类型。

```plain
type add func(a int, b int) int  
```

上面的代码片段创建了一个新的函数类型`add`，它接受两个整数参数并返回一个整数。现在我们可以定义`add`类型的变量。

让我们编写一个定义`add`类型变量的程序。
```golang
package main

import (  
    "fmt"
)

type add func(a int, b int) int

func main() {  
    var a add = func(a int, b int) int {
        return a + b
    }
    s := a(5, 6)
    fmt.Println("Sum", s)
}
```
[Run in playground](https://play.golang.org/p/n3yPQ7hG7ip)

在上面的程序的第 10 行，我们定义了一个类型为`add`的变量`a`，并为其赋予与`add`类型匹配的函数。在第 13 行，我们调用该函数，并将结果分配给`s`。该程序将打印，
```plain
Sum 11  
```

## 高阶函数

[wiki]([https://en.wikipedia.org/wiki/Higher-order_function](https://en.wikipedia.org/wiki/Higher-order_function)
)上高阶函数的定义是满足以下至少一个条件

- 将一个或多个函数作为参数
- 返回一个函数作为结果

让我们看一下上述两种情况的一些简单示例。

##### 将函数作为参数传递给其他函数

```golang
package main

import (  
    "fmt"
)

func simple(a func(a, b int) int) {  
    fmt.Println(a(60, 7))
}

func main() {  
    f := func(a, b int) int {
        return a + b
    }
    simple(f)
}
```
[Run in playground](https://play.golang.org/p/C0MNwz2TSGU)

在上面的例子中的第 7 行，我们定义一个函数`simple`，它接受一个函数，该函数接受两个`int`参数并返回一个`int`作为参数。在 main 函数里面，我们创建了一个匿名函数`f`，其与函数`simple`的参数匹配。我们调用`simple`，并在下一行中将`f`作为参数传递给它。该程序打印`67`作为输出。

##### 从其他函数返回函数
现在让我们重写上面的程序并从`simple`函数返回一个函数。

```golang
package main

import (  
    "fmt"
)

func simple() func(a, b int) int {  
    f := func(a, b int) int {
        return a + b
    }
    return f
}

func main() {  
    s := simple()
    fmt.Println(s(60, 7))
}
```

[Run in playground](https://play.golang.org/p/82y2caejUy8)

在上面程序中的第 7 行，`simple`函数返回一个函数，该函数接受两个`int`参数并返回一个`int`参数。

在第 15 行调用了`simple`函数，然后将返回值分配给`s`。现在`s`包含`simple`函数返回的函数。我们调用`s`并传递两个`int`参数。 该程序将输出`67`。

## 闭包

闭包是匿名函数的特例。闭包是匿名函数，用于访问函数体外定义的变量。

用一个例子在了解它，

```golang
package main

import (  
    "fmt"
)

func main() {  
    a := 5
    func() {
        fmt.Println("a =", a)
    }()
}
```
[Run in playground](https://play.golang.org/p/6QriMs-zbnf)


在上面程序中的第 10 行，匿名函数访问函数体外定义的变量`a`。因此，这个匿名函数是一个闭包。

每个闭包都绑定到它自己的周围变量。让我们通过一个简单的例子来理解这是什么意思。

```golang
package main

import (  
    "fmt"
)

func appendStr() func(string) string {  
    t := "Hello"
    c := func(b string) string {
        t = t + " " + b
        return t
    }
    return c
}

func main() {  
    a := appendStr()
    b := appendStr()
    fmt.Println(a("World"))
    fmt.Println(b("Everyone"))

    fmt.Println(a("Gopher"))
    fmt.Println(b("!"))
}
```

[Run in playground](https://play.golang.org/p/134NiQGPOcS)

在上面的程序中，函数`appendStr`返回一个闭包。该闭包绑定到变量`t`。

变量`a`和`b`分别在第 17 和 18 行中声明为闭包，它们与自己的`t`值绑定。

我们先用参数`World`调用`a`。现在，一个版本的`t`的值变成了`Hello World`。

在第 20 行，我们用参数`Everyone`调用`b`。由于`b`绑定到自己的变量`t`，因此`b`的`t`版本的初始值为`Hello`。因此，在此函数调用之后，`b`的`t`版本的值变为`Hello Everyone`。该程序剩下的功能都是不难理解的。

程序将输出，
```plain
Hello World  
Hello Everyone  
Hello World Gopher  
Hello Everyone ! 
```

## 头等函数的实际用法

到目前为止，我们已经定义了头等函数，并且我们已经看到了一些人为的例子来了解它们的工作原理。现在让我们编写一个具体的程序，它揭示了头等函数的实际用法。

我们将创建一个程序，根据某些标准过滤一部分学生。让我们一步一步地解决这个问题。

首先定义学生类型，
```golang
type student struct {  
    firstName string
    lastName string
    grade string
    country string
}
```

下一步是编写`filter`函数。该函数的入参为`student`结构的切片和一个确认`student`是符合过滤条件的函数。一旦我们编写这个函数，我们会更好地理解。让我们继续吧。
```golang
func filter(s []student, f func(student) bool) []student {  
    var r []student
    for _, v := range s {
        if f(v) == true {
            r = append(r, v)
        }
    }
    return r
}
```

在上面的函数中，第二个参数是一个以`student`为参数并返回`bool`的函数。此函数是为了确认该`student`是否符合条件。我们在第 3 行中迭代`student`切片。我们将每个`student`作为参数传递给函数`f`。如果返回`true`，则表示`student`已通过过滤条件，并将其添加到结果切片`r`中。您可能对此功能的实际使用感到有些困惑，但是一旦我们完成该程序就会很清楚。我添加了 main 函数，并在下面提供了完整的程序。

```golang
package main

import (  
    "fmt"
)

type student struct {  
    firstName string
    lastName  string
    grade     string
    country   string
}

func filter(s []student, f func(student) bool) []student {  
    var r []student
    for _, v := range s {
        if f(v) == true {
            r = append(r, v)
        }
    }
    return r
}

func main() {  
    s1 := student{
        firstName: "Naveen",
        lastName:  "Ramanathan",
        grade:     "A",
        country:   "India",
    }
    s2 := student{
        firstName: "Samuel",
        lastName:  "Johnson",
        grade:     "B",
        country:   "USA",
    }
    s := []student{s1, s2}
    f := filter(s, func(s student) bool {
        if s.grade == "B" {
            return true
        }
        return false
    })
    fmt.Println(f)
}
```

[Run in playground](https://play.golang.org/p/YUL1CqSrvfc)


在 main 函数中，我们首先创建两个`student`类型的变量`s1`和`s2`，并将它们添加到切片`s`中作为参数传递给`filter`函数。现在我想找出所有`B`级的学生。我们在上面的程序中通过一个函数来检查学生是否有`B`级，如果是，那么返回`true`。上述程序将打印，

```plain
[{Samuel Johnson B USA}]
```


假设我们想找到所有来自印度的学生。将`filter`函数参数改变一下，可以轻松完成此操作。
我提供了以下代码，
```golang
c := filter(s, func(s student) bool {  
    if s.country == "India" {
        return true
    }
    return false
})
fmt.Println(c)  
```

请将其添加到 main 函数并检查输出。

让我们再写一个程序来结束本节。该程序将对切片的每个元素执行相同的操作并返回结果。例如，如果我们想要将切片中的所有整数乘以 5 并返回输出，则可以使用头等函数轻松完成。这些对集合的每个元素进行操作的函数称为`map`函数。我已经提供了以下程序。

```golang
package main

import (  
    "fmt"
)

func iMap(s []int, f func(int) int) []int {  
    var r []int
    for _, v := range s {
        r = append(r, f(v))
    }
    return r
}
func main() {  
    a := []int{5, 6, 7, 8, 9}
    r := iMap(a, func(n int) int {
        return n * 5
    })
    fmt.Println(r)
}
```

上述程序将输出，
```plain
[25 30 35 40 45]
```

快速回顾一下本教程的内容，

- 什么是头等函数
- 匿名函数
- 用户定义的函数类型
- 高阶函数
  - 将函数作为参数传递给其他函数
  - 从其他函数返回函数
- 闭包
- 头等函数的实际用法





