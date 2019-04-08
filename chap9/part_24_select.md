> * 原文地址：[Part 24: Select](https://golangbot.com/select/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

## 什么是 select

````select````语句用于从多个发送/接收```channel```中进行选择的操作。 ```select```语句将阻塞直到其中一个发送/接收操作准备就绪。如果有多个操作就绪，则随机选择其中一个操作。语法类似于```switch```，只是每个```case```语句被一个`channel`操作取代了。让我们深入研究一些代码，以便更好地理解
```golang
package main

import (
    "fmt"
    "time"
)

func server1(ch chan string) {
    time.Sleep(6 * time.Second)
    ch <- "from server1"
}
func server2(ch chan string) {
    time.Sleep(3 * time.Second)
    ch <- "from server2"

}
func main() {
    output1 := make(chan string)
    output2 := make(chan string)
    go server1(output1)
    go server2(output2)
    select {
    case s1 := <-output1:
        fmt.Println(s1)
    case s2 := <-output2:
        fmt.Println(s2)
    }
}
```
[Run in playgroud](https://play.golang.org/p/3_yaJSoSpG)

在上面的程序中，在第 8 行`server1`函数休眠 6 秒然后将文本从`server1`写入`channel ch`。第 12 行```server2```函数休眠 3 秒，然后从```server2```写入```channel ch```。

`main`函数在 20 和 21 行分别调用`server1`和`server2`。

在第 22 行，`select`语句将阻塞直到其中一个`case`准备就绪。在上面的程序中，`server1`在 6 秒后写入```output1 channel```，而```server2```在 3 秒后写入```output2 channel```。因此 select 语句将阻塞 3 秒并等待```server2```写入。 3 秒后，程序将打印，
```plain
from server2
```
然后终止。

## `select` 的用途

将上述程序中的函数命名为`server1`和`server2`的原因是为了说明`select`的实际用途。

让我们假设我们有一个关键任务的应用，我们需要尽快将输出返回给用户。该应用程序的数据库被复制并存储在世界各地的不同服务器中。假设函数`server1`和`server2`实际上与 2 个这样的服务器通信。每个服务器的响应时间取决于每个服务器的负载和网络延迟。我们将请求发送到两个服务器，然后使用`select`语句在相应的`channel`上等待响应。`select`会选择优先响应的服务器，其他响应被忽略。这样我们就可以向多个服务器发送相同的请求，并将最快的响应返回给用户:)。

## 默认`case`

当其他`case`都没有准备就绪时，将会执行`select`语句中的默认`case`。这通常用于防止`select`语句阻塞。
```golang
package main

import (
    "fmt"
    "time"
)

func process(ch chan string) {
    time.Sleep(10500 * time.Millisecond)
    ch <- "process successful"
}

func main() {
    ch := make(chan string)
    go process(ch)
    for {
        time.Sleep(1000 * time.Millisecond)
        select {
        case v := <-ch:
            fmt.Println("received value: ", v)
            return
        default:
            fmt.Println("no value received")
        }
    }

}
```
[Run in playground](https://play.golang.org/p/8xS5r9g1Uy)

在上面的程序中，在第 8 行`process`函数休眠 10500 毫秒（10.5 秒），然后将```process successful```写入`ch channel`。该函数在第 15 行被并发调用。

在并发调用`process Goroutine`之后，`main Goroutine`中启动了无限循环。无限循环在每次迭代开始期间休眠 1000 毫秒（1 秒），并执行`select`操作。在前 10500 毫秒期间，`select`语句的第一种情况即`case v：= <-ch：`将不会准备就绪，因为`process Goroutine`仅在 10500 毫秒后才写入`ch channel`。因此，在此期间将执行`defualt`分支，程序将会打印 10 次`no value received`。

在 10.5 秒之后，`process Goroutine`将`process successful`写入`ch`。 现在将执行`select`语句的第一种情况，程序将打印`received value:  process successful`然后程序终止。该程序将输出，
```plain
no value received
no value received
no value received
no value received
no value received
no value received
no value received
no value received
no value received
no value received
received value:  process successful
```
## 死锁和默认`case`
```golang
package main

func main() {
    ch := make(chan string)
    select {
    case <-ch:
    }
}
```
[Run in playgroud](http://play.flysnow.org/p/sZj8dfIjpfW)

在上面的程序中，我们在第一行创建了一个`channel ch`。我们尝试从选择的这个`channel`读取。而这个`select`语句将一直阻塞，因为没有其他`Goroutine`写入此`channel`，因此将导致死锁。该程序将在运行时产生`panic`同时打印，
```plain
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:
main.main()
    /tmp/sandbox416567824/main.go:6 +0x80
```

如果存在默认`case`，则不会发生此死锁，因为在没有其他`case`准备就绪时将执行默认`case`。上面的程序可以重写。
```golang
package main

import "fmt"

func main() {
    ch := make(chan string)
    select {
    case <-ch:
    default:
        fmt.Println("default case executed")
    }
}
```
[Run in playground](https://play.golang.org/p/Pxsh_KlFUw)

输出，
```plain
default case executed
```

类似地，当`select`只有一个`nil channel`，也会执行默认`case`。

```golang
package main

import "fmt"

func main() {
    var ch chan string
    select {
    case v := <-ch:
        fmt.Println("received value", v)
    default:
        fmt.Println("default case executed")

    }
}
```
[Run in playground](https://play.golang.org/p/IKmGpN61m1)

在上面的程序中，`ch`是`nil`，我们试图用`select`从`ch`中读取。如果没有默认`case`，则`select`将一直被阻塞并导致死锁。由于我们在`select`中有一个默认的`case`，它将被执行并且程序将打印，
```plain
default case executed
```
## `select`的随机性

当`select`语句中的多个`case`准备就绪时，将会随机挑选一个执行。

```golang
package main

import (
    "fmt"
    "time"
)

func server1(ch chan string) {
    ch <- "from server1"
}
func server2(ch chan string) {
    ch <- "from server2"

}
func main() {
    output1 := make(chan string)
    output2 := make(chan string)
    go server1(output1)
    go server2(output2)
    time.Sleep(1 * time.Second)
    select {
    case s1 := <-output1:
        fmt.Println(s1)
    case s2 := <-output2:
        fmt.Println(s2)
    }
}
```
[Run in playground](https://play.golang.org/p/vJ6VhVl9YY)

在上面的程序中，`server1`和`server2` 协程在第 18 和 19 行分别被调用，然后```main```协程休眠 1 秒。当运行到`select`语句时，`server1`已将`from server1`写入`output1`，`server2`已将`from server2`写入`output2`，因此`select`语句中的两种情况都准备就绪。如果多次运行此程序，将会随机输出`from server1`或`from server2`。

## 空`select`
```golang
package main

func main() {
    select {}
}
```
[Run in playground](https://play.golang.org/p/u8hErIxgxs)

你认为上面的程序将会输出什么？

我们知道`select`语句将被阻塞，直到执行其中一个`case`。在这种情况下，`select`语句没有任何`case`，因此它将一直阻塞导致死锁。这个程序将会产生`panic`，并输出，
```plain
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [select (no cases)]:
main.main()
    /tmp/sandbox299546399/main.go:4 +0x20
```

