> * 原文地址：[Part 32: Panic and Recover](https://golangbot.com/panic-and-recover/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

## 什么是 panic
处理 Go 中异常情况的惯用方法是使用 errors，对于程序中出现的大多数异常情况，errors 就足够了。

但是在某些情况下程序不能在异常情况下继续正常执行。在这种情况下，我们使用 panic 来终止程序。函数遇到 panic 时将会停止执行，如果有 defer 的话就执行 defer 延迟函数，然后返回其调用者。此过程一直持续到当前 goroutine 的所有函数都返回，然后打印出 panic 信息，然后是堆栈信息，然后程序终止。待会儿用一个例子来解释，这个概念就会更加清晰一些了。

我们可以使用 recover 函数恢复被 panic 终止的程序，将在本教程后面讨论。

panic 和 recover 有点类似于其他语言中的 try-catch-finally 语句，但是前者使用的比较少，而且使用时更优雅代码也更简洁。

## 什么时候应该用 panic

一般情况下我们应该避免使用 panic 和 recover，尽可能使用 errors。只有在程序无法继续执行的情况下才应该使用 panic 和 recover。

##### 两个 panic 典型应用场景

1. 不可恢复的错误，让程序不能继续进行。
比如说 Web 服务器无法绑定到指定端口。在这种情况下，panic 是合理的，因为如果端口绑定失败接下来的逻辑继续也是没有意义的。

2. coder 的人为错误
假设我们有一个接受指针作为参数的方法，然而使用了 nil 作为参数调用此方法。在这种情况下，我们可以用 panic，因为该方法需要一个有效的指针。

## panic 示例
panic 函数的定义
```golang
func panic(interface{})
```
当程序终止时，参数会传递给 panic 函数打印出来。看看下面例子的 panic 是如何使用的。
```golang
package main

import (
    "fmt"
)

func fullName(firstName *string, lastName *string) {
    if firstName == nil {
        panic("runtime error: first name cannot be nil")
    }
    if lastName == nil {
        panic("runtime error: last name cannot be nil")
    }
    fmt.Printf("%s %s\n", *firstName, *lastName)
    fmt.Println("returned normally from fullName")
}

func main() {
    firstName := "Elon"
    fullName(&firstName, nil)
    fmt.Println("returned normally from main")
}
```
[Run in playground](http://play.flysnow.org/p/peylhn6NBMH)

上面这段代码，fullName 函数功能是打印一个人的全名。此函数检查 firstName 和 lastName 指针是否为 nil。如果它为 nil，则函数调用 panic 并显示相应的错误消息。程序终止时将打印此错误消息和错误堆栈信息。

运行此程序将打印以下输出，
```plain
panic: runtime error: last name cannot be nil
goroutine 1 [running]:
main.fullName(0x1040c128, 0x0)
    /tmp/sandbox135038844/main.go:12 +0x120
main.main()
    /tmp/sandbox135038844/main.go:20 +0x80
```

我们来分析一下这个输出，来了解 panic 是如何工作以及如何打印堆栈跟踪的。
在第 19 行，我们将 Elon 定义给 firstName。然后调用 fullName 函数，其中 lastName 参数为 nil。因此，第 11 行将触发 panic。当触发 panic 时，程序执行就终止了，然后打印传递给 panic 的内容，最后打印堆栈跟踪信息。因此 14 行以后的代码不会被执行。
该程序首先打印传递给 panic 函数的内容，
```plain
panic: runtime error: last name cannot be nil
```
然后打印堆栈跟踪信息。
该程序在 12 行触发 panic，因此，
```plain
ain.fullName(0x1040c128, 0x0)
    /tmp/sandbox135038844/main.go:12 +0x120
```
将被首先打印。然后将打印堆栈中的下一个内容，
```plain
main.main()
    /tmp/sandbox135038844/main.go:20 +0x80
```
现在已经返回到了造成 panic 的顶层 main 函数，因此打印结束。

## defer 函数
我们回想一下 panic 的作用。当函数遇到 panic 时，将会终止 panic 后面代码的执行，如果函数体包含有 defer 函数的话会执行完 defer 函数。然后返回其调用者。此过程一直持续到当前 goroutine 的所有函数都返回，此时程序打印出 panic 内容，然后是堆栈跟踪信息，然后终止。

在上面的示例中，我们没有任何 defer 函数的调用。修改下上面的例子，来看看 defer 函数的例子吧。
```golang
package main

import (
    "fmt"
)

func fullName(firstName *string, lastName *string) {
    defer fmt.Println("deferred call in fullName")
    if firstName == nil {
        panic("runtime error: first name cannot be nil")
    }
    if lastName == nil {
        panic("runtime error: last name cannot be nil")
    }
    fmt.Printf("%s %s\n", *firstName, *lastName)
    fmt.Println("returned normally from fullName")
}

func main() {
    defer fmt.Println("deferred call in main")
    firstName := "Elon"
    fullName(&firstName, nil)
    fmt.Println("returned normally from main")
}
```
[Run in playground](http://play.flysnow.org/p/0N8J6AvTObI
)
对之前代码所做的唯一更改是在 fullName 函数和 main 函数中第一行添加了 defer 函数调用。
运行的输出，
```plain
deferred call in fullName
deferred call in main
panic: runtime error: last name cannot be nil

goroutine 1 [running]:
main.fullName(0x1042bf90, 0x0)
    /tmp/sandbox060731990/main.go:13 +0x280
main.main()
    /tmp/sandbox060731990/main.go:22 +0xc0
```
当发生 panic 时，首先执行 defer 函数，然后到下一个 defer 调用，依此类推，直到达到顶层调用者。

在我们的例子中，defer 声明在 fullName 函数的第一行。首先执行 fullName 函数。打印
```plain
deferred call in fullName
```

然后调用返回到 main 函数的 defer，
```plain
deferred call in main
```
现在调用已返回到顶层函数，然后程序打印 panic 内容，然后是堆栈跟踪信息，然后终止。

## recover 函数
recover 是一个内置函数，用于 goroutine 从 panic 的中断状况中恢复。
函数定义如下，
```golang
func recover() interface{}
```
recover 只有在 defer 函数内部调用时才有效。defer 函数内通过调用 recover 可以让 panic 中断的程序恢复正常执行，调用 recover 会返回 panic 的内容。如果在 defer 函数之外调用 recover，它将不会停止 panic 序列。

修改一下，使用 recover 来让 panic 恢复正常执行。
```golang
package main

import (
    "fmt"
)

func recoverName() {
    if r := recover(); r!= nil {
        fmt.Println("recovered from ", r)
    }
}

func fullName(firstName *string, lastName *string) {
    defer recoverName()
    if firstName == nil {
        panic("runtime error: first name cannot be nil")
    }
    if lastName == nil {
        panic("runtime error: last name cannot be nil")
    }
    fmt.Printf("%s %s\n", *firstName, *lastName)
    fmt.Println("returned normally from fullName")
}

func main() {
    defer fmt.Println("deferred call in main")
    firstName := "Elon"
    fullName(&firstName, nil)
    fmt.Println("returned normally from main")
}
```
[Run in playground](http://play.flysnow.org/p/k9oZlqpkVKf)
第 7 行调用了 recoverName 函数。这里打印了 recover 返回的值，
发现 recover 返回的是 panic 的内容。

打印如下，
```plain
recovered from  runtime error: last name cannot be nil
returned normally from main
deferred call in main
```
程序在 19 行触发 panic，defer 函数 recoverName 通过调用 recover 来重新控制该 goroutine，
```plain
recovered from  runtime error: last name cannot be nil
```
在执行 recover 之后，panic 停止并且返回到调用者，main 函数和程序在触发 panic 之后将继续从第 29 行执行。然后打印，
```plain
returned normally from main
deferred call in main
```
## Panic, Recover 和 Goroutines

recover 仅在从同一个 goroutine 调用时才起作用。从不同的 goroutine 触发的 panic 中 recover 是不可能的。再来一个例子来加深理解。
```golang
package main

import (
    "fmt"
    "time"
)

func recovery() {
    if r := recover(); r != nil {
        fmt.Println("recovered:", r)
    }
}

func a() {
    defer recovery()
    fmt.Println("Inside A")
    go b()
    time.Sleep(1 * time.Second)
}

func b() {
    fmt.Println("Inside B")
    panic("oh! B panicked")
}

func main() {
    a()
    fmt.Println("normally returned from main")
}
```
[Run in playground](http://play.flysnow.org/p/-0yB_Ja_UcZ)
在上面的程序中，函数 b 在 23 行触发 panic。函数 a 调用 defer 函数 recovery 用于从 panic 中恢复。函数 a 的 17 行用另外一个 goroutine 执行 b 函数。Sleep 的作用只是为了确保程序在 b 运行完毕之前不会被终止，当然也可以用 sync.WaitGroup 来解决。

你认为该段代码的输出是什么？panic 会被恢复吗？答案是不可以。panic 将无法被恢复。这是因为 recover 存在于不同的 gouroutine 中，并且触发 panic 发生在不同 goroutine 执行的 b 函数。因此无法恢复。
运行的输出，
```plain
Inside A
Inside B
panic: oh! B panicked

goroutine 5 [running]:
main.b()
    /tmp/sandbox388039916/main.go:23 +0x80
created by main.a
    /tmp/sandbox388039916/main.go:17 +0xc0
```

可以从输出中看到恢复失败了。

如果在同一个 goroutine 中调用函数 b，那么 panic 就会被恢复。
在第 17 行把， ```go b()``` 换成 ```b()```
那么会输出，
```plain
Inside A
Inside B
recovered: oh! B panicked
normally returned from main
```

## 运行时的 panic

panic 还可能由运行时的错误引起，例如数组越界访问。这相当于使用由接口类型[runtime.Error](https://golang.org/src/runtime/error.go?s=267:503#L1)定义的参数调用内置函数 panic。 runtime.Error 接口的定义如下，
```plain
type Error interface {
    error
    // RuntimeError is a no-op function but
    // serves to distinguish types that are run time
    // errors from ordinary errors: a type is a
    // run time error if it has a RuntimeError method.
    RuntimeError()
}
```
runtime.Error 接口满足内置接口类型[error](https://golangbot.com/error-handling/#errortyperepresentation)。

让我们写一个人为的例子来创建运行时 panic。
```golang
package main

import (
    "fmt"
)

func a() {
    n := []int{5, 7, 4}
    fmt.Println(n[3])
    fmt.Println("normally returned from a")
}
func main() {
    a()
    fmt.Println("normally returned from main")
}
```
[Run in playground](http://play.flysnow.org/p/L8i-JdQz4SR)
在上面的程序中，第 9 行我们试图访问 n [3]，这是切片中的无效索引。这个会触发 panic，输出如下，
```plain
panic: runtime error: index out of range

goroutine 1 [running]:
main.a()
    /tmp/sandbox780439659/main.go:9 +0x40
main.main()
    /tmp/sandbox780439659/main.go:13 +0x20
```
您可能想知道是否运行中的 panic 能够被恢复。答案是肯定的。让我们修改上面的程序，让 panic 恢复过来。
```golang
package main

import (
    "fmt"
)

func r() {
    if r := recover(); r != nil {
        fmt.Println("Recovered", r)
    }
}

func a() {
    defer r()
    n := []int{5, 7, 4}
    fmt.Println(n[3])
    fmt.Println("normally returned from a")
}

func main() {
    a()
    fmt.Println("normally returned from main")
}
```
[Run in playground](http://play.flysnow.org/p/i-9AA33FiBn)
执行后输出，
```plain
Recovered runtime error: index out of range
normally returned from main
```
显然可以看到 panic 被恢复了。

## recover 后获取堆栈信息
我们恢复了 panic，但是丢失了这次 panic 的堆栈调用的信息。
有一种方法可以解决这个，就是使用 Debug 包中的 PrintStack 函数打印堆栈跟踪信息
```golang
package main

import (
    "fmt"
    "runtime/debug"
)

func r() {
    if r := recover(); r != nil {
        fmt.Println("Recovered", r)
        debug.PrintStack()
    }
}

func a() {
    defer r()
    n := []int{5, 7, 4}
    fmt.Println(n[3])
    fmt.Println("normally returned from a")
}

func main() {
    a()
    fmt.Println("normally returned from main")
}
```
[Run in playground](http://play.flysnow.org/p/H-GRDqoE1jU)
在 11 行调用了 debug.PrintStack，可以看到随后输出，
```plain
Recovered runtime error: index out of range
goroutine 1 [running]:
runtime/debug.Stack(0x1042beb8, 0x2, 0x2, 0x1c)
    /usr/local/go/src/runtime/debug/stack.go:24 +0xc0
runtime/debug.PrintStack()
    /usr/local/go/src/runtime/debug/stack.go:16 +0x20
main.r()
    /tmp/sandbox949178097/main.go:11 +0xe0
panic(0xf0a80, 0x17cd50)
    /usr/local/go/src/runtime/panic.go:491 +0x2c0
main.a()
    /tmp/sandbox949178097/main.go:18 +0x80
main.main()
    /tmp/sandbox949178097/main.go:23 +0x20
normally returned from main
```

从输出中可以知道，首先是 panic 被恢复然后打印```Recovered runtime error: index out of range```，再然后打印堆栈跟踪信息。最后在 panic 被恢复后打印```normally returned from main```
