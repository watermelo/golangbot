> * 原文地址：[Part 12: Variadic Functions]([https://golangbot.com/variadic-functions/](https://golangbot.com/variadic-functions/)
)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

## 什么是变参函数
变参函数是一个可以接受参数数量可变的函数。

## 语法

如果函数的最后一个参数用`...T`表示，那么该函数最后一个参数可以接受任意数量的`T`类型的参数。

*注意，只允许函数的最后一个参数为可变参数。*

## 示例
`append`函数可以增加任意数量的参数给切片。变参函数就是用到了切片的这个原理。

```golang
func append(slice []Type, elems ...Type) []Type  
```

以上是`append`函数的定义。在这个定义中，`elems`是一个可变参数。通过这个语法，`append`可以接受可变数量的参数。

来创建一个变参函数。我们将编写一个查找输入列表中是否存在指定整数的简单程序。

```golang
package main

import (  
    "fmt"
)

func find(num int, nums ...int) {  
    fmt.Printf("type of nums is %T\n", nums)
    found := false
    for i, v := range nums {
        if v == num {
            fmt.Println(num, "found at index", i, "in", nums)
            found = true
        }
    }
    if !found {
        fmt.Println(num, "not found in ", nums)
    }
    fmt.Printf("\n")
}
func main() {  
    find(89, 89, 90, 95)
    find(45, 56, 67, 45, 90, 109)
    find(78, 38, 56, 98)
    find(87)
}
```
[Run in playground](https://play.golang.org/p/7occymiS6s)

在上面的程序中，`func find(num int, nums ...int)`接收可变数量的参数`nums`。在函数`find`中，`nums`的类型等价于`[]int`，即整数切片。

变参函数的原理是将传递的可变参数转换为一个新切片。例如，在上面程序的第 22 中，`find`函数的可变参数是 89, 90, 95。`find`函数需要一个可变的`int`参数。因此，这三个参数将由编译器转换为 int 型的`[] int {89,90,95}`切片，然后它将被传递给`find`函数。

上述程序将打印,
```plain
type of nums is []int  
89 found at index 0 in [89 90 95]

type of nums is []int  
45 found at index 2 in [56 67 45 90 109]

type of nums is []int  
78 not found in  [38 56 98]

type of nums is []int  
87 not found in  []  
```

上面程序的第 25 行，`find`函数调用只有一个参数。我们没有向可变参数传递任何值。这是完全合法的，在这种情况下，`nums`是一个长度和容量都为 0 的切片。

## 使用切片作为变参函数的参数
我们将一个切片传递给一个变参函数，并看看下面的例子会发生什么。

```golang
package main

import (  
    "fmt"
)

func find(num int, nums ...int) {  
    fmt.Printf("type of nums is %T\n", nums)
    found := false
    for i, v := range nums {
        if v == num {
            fmt.Println(num, "found at index", i, "in", nums)
            found = true
        }
    }
    if !found {
        fmt.Println(num, "not found in ", nums)
    }
    fmt.Printf("\n")
}
func main() {  
    nums := []int{89, 90, 95}
    find(89, nums)
}
```
[Run in playground](https://play.golang.org/p/A-DNilpH2L)


在第 23 行，我们将一个切片传递给一个变参函数。

上面的程序将会出现编译错误`main.go:23: cannot use nums (type []int) as type int in argument to find`

为什么这个例子不能运行呢？因为`find`函数的签名如下所示，

```golang
func find(num int, nums ...int) 
```

根据变参函数的定义，`nums ...int`意味着它将接受 int 类型的可变数量的参数。

在第 23 行中，`nums`作为可变参数传递给`find`函数。正如我们已经讨论过的，这些可变参数将被转换为 int 类型的切片，因为 find 需要可变的 int 参数。在这个例子中，`nums`已经是一个 int 切片了，并且还尝试使用 nums 去创建切片，即编译器尝试执行

```golang
find(89, []int{nums}) 
```

因为 nums 是`[] int`而不是 int，所以会失败。

那么有没有办法将切片传递给可变参数函数？当然可以。

有一个语法糖可用于将切片传递给变参函数。就是在切片后使用`...`。如果这样做，切片将直接传递给函数，而不会创建新切片。

上述代码中，我们用`find(89, nums...)`替换`find(89, nums)`，程序将运行并打印，
```plain
type of nums is []int  
89 found at index 0 in [89 90 95]  
```

这是完整的程序供参考。

```golang
package main

import (  
    "fmt"
)

func find(num int, nums ...int) {  
    fmt.Printf("type of nums is %T\n", nums)
    found := false
    for i, v := range nums {
        if v == num {
            fmt.Println(num, "found at index", i, "in", nums)
            found = true
        }
    }
    if !found {
        fmt.Println(num, "not found in ", nums)
    }
    fmt.Printf("\n")
}
func main() {  
    nums := []int{89, 90, 95}
    find(89, nums...)
}
```
[Run inplayground](https://play.golang.org/p/IvzwhzhFsT)

## 注意事项
当你在变参函数中修改切片时，请确保知道自己在做什么。

来看个例子。

```golang

package main

import (  
    "fmt"
)

func change(s ...string) {  
    s[0] = "Go"
}

func main() {  
    welcome := []string{"hello", "world"}
    change(welcome...)
    fmt.Println(welcome)
}
```
[Run in playground](https://play.golang.org/p/R0GsuW7rdd)

你觉得上面的代码会输出什么？如果你觉得是 `[Go world]`，那么恭喜你！你已经理解了变参函数和切片。如果你弄错了，没什么大不了的，让我解释一下我们为什么是这个输出。

上面程序的第 13 行，我们使用语法糖`...`并将切片作为可变参数传递给`change`函数。

正如我们已经讨论的那样，如果使用了`...`，`welcome`切片本身将作为参数传递，而不会创建新的切片。因此`welcome`将作为参数传递给`change`函数。

在`change`函数内部，切片的第一个元素更改为 Go。因此该程序输出

```plain
[Go world]
```

这是另一个了解变参函数的程序。

```golang
package main

import (  
    "fmt"
)

func change(s ...string) {  
    s[0] = "Go"
    s = append(s, "playground")
    fmt.Println(s)
}

func main() {  
    welcome := []string{"hello", "world"}
    change(welcome...)
    fmt.Println(welcome)
}
```
[Run in playground](https://play.golang.org/p/WdbFIkdLoe)

这个作为练习让你弄清楚上面的程序是如何工作的:)。





