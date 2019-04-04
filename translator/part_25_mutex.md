> * 原文地址：[Part 25: Mutex](https://golangbot.com/mutex/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

在本教程中，我们将了解互斥锁`Mutex`。我们还将学习如何使用`Mutex`和`channel`解决竞争条件。

## 临界区

在了解互斥锁之前，先了解并发编程中[临界区](https://en.wikipedia.org/wiki/Critical_section)的概念非常重要。当程序并发运行时，多个`Goroutines`不应该同时拥有修改共享内存的权限。修改共享内存的这部分代码则称为临界区。例如，假设我们有一段代码将变量 x 递增 1。
```golang
x = x + 1  
```
如果上面一段代码被一个`Goroutine`访问，就不会有任何问题。

让我们看看为什么当有多个`Goroutines`并发运行时，这段代码会失败。为简单起见，我们假设我们有 2 个`Goroutines`并发运行上面的代码行。

上面的代码将按以下步骤执行（有更多技术细节涉及寄存器，如何添加任务等等，但为了本教程的简便，我们假设都是第三步），

> 1. 获取当前`x`的值
> 2. 计算 `x + 1`
> 3. 把第二步计算的值赋给`x`

当这三个步骤仅由一个`Goroutine`进行时，结果没什么问题。

让我们看看当两个`Goroutines`并发运行此代码时会发生什么。下图描绘了当两个`Goroutines`并发访问代码行`x = x + 1`时可能发生的情况。

![](https://upload-images.jianshu.io/upload_images/8573331-6be57cffd3b8baa7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们假设`x`的初始值为 0。`Goroutine 1`获取`x`的初始值，计算`x + 1`，在它将计算值赋值给`x`之前，系统切换到`Goroutine 2`。现在`Goroutine 2`获取的`x`的值仍为 0，然后计算`x + 1`。此时系统再次切回到`Goroutine 1`。现在`Goroutine 1`将其计算值 1 赋值给`x`，因此`x`变为 1。然后`Goroutine 2`再次开始执行然后赋值计算值，然后把 1 赋值给`x`，因此在两个`Goroutines`执行后`x`为 1。

现在让我们看看可能发生的不同情况。

![](https://user-gold-cdn.xitu.io/2019/4/4/169e66f0c7b9c938?w=512&h=702&f=png&s=45194)

在上面的场景中，`Goroutine 1`开始执行并完成所有的三个步骤，因此`x`的值变为 1。然后`Goroutine 2`开始执行。现在`x`的值为 1，当`Goroutine 2`完成执行时，`x`的值为 2。

因此，在这两种情况下，可以看到`x`的最终值为 1 或者 2，具体取决于协程如何切换。这种程序的输出取决于`Goroutines`的执行顺序的情况，称为[竞态条件](https://en.wikipedia.org/wiki/Race_condition)。

在上面的场景中，如果在任何时间点只允许一个`Goroutine`访问临界区，则可以避免竞态条件。这可以通过使用 Mutex 实现。

## `Mutex`互斥锁
`Mutex`用于提供锁定机制，以确保在任何时间点只有一个`Goroutine`运行代码的临界区，以防止发生竞态条件。

`sync`包中提供了`Mutex`。`Mutex`上定义了两种方法，即锁定`Lock`和`UnLock`。在`Lock`和`UnLock`的调用之间的任何代码将只能由一个`Goroutine`执行，从而避免竞态条件。
```plain
mutex.Lock()  
x = x + 1  
mutex.Unlock()  
```

在上面的代码中，`x = x + 1`将仅由一个`Goroutine`执行。

如果一个`Goroutine`已经持有锁，当一个新的`Goroutine`试图获取锁的时候，新的`Goroutine`将被阻塞直到互斥锁被解锁。

## 拥有竞态条件的程序

在本节中，我们将编写一个有竞态条件的程序，在接下来的部分中我们将修复竞态条件。
```golang
package main  
import (  
    "fmt"
    "sync"
    )
var x  = 0  
func increment(wg *sync.WaitGroup) {  
    x = x + 1
    wg.Done()
}
func main() {  
    var w sync.WaitGroup
    for i := 0; i < 1000; i++ {
        w.Add(1)        
        go increment(&w)
    }
    w.Wait()
    fmt.Println("final value of x", x)
}
```
[Run in playgroud](https://play.golang.org/p/q6ONK-wP87B)

在上面的程序中，第 7 行的`increment`函数将`x`的值递增 1，然后调用`WaitGroup`上的`Done`以通知`main Goroutine` 任务完成。

我们在第 15 行生成 1000 个`increment Goroutines`。这些`Goroutines`中的每一个都并发运行，当多个`Goroutines`尝试同时访问`x`的值，并且计算`x + 1`时会出现竞态条件。

最好在本地运行此程序，因为`playgroud`是确定性的不会出现竞态条件。在本地计算机上多次运行此程序，您可以看到由于竞态条件，每次输出都会不同。我遇到的一些输出是`final value of x 941, final value of x 928, final value of x 922`等等。

## 使用互斥锁 `Mutex` 解决竞态条件

在上面的程序中，我们生成了 1000 个`Goroutines`。如果每个都将`x`的值加 1，则`x`的最终期望值应该为 1000。在本节中，我们将使用互斥锁 `Mutex` 修复上述程序中的竞态条件。
```golang
package main  
import (  
    "fmt"
    "sync"
    )
var x  = 0  
func increment(wg *sync.WaitGroup, m *sync.Mutex) {  
    m.Lock()
    x = x + 1
    m.Unlock()
    wg.Done()
}
func main() {  
    var w sync.WaitGroup
    var m sync.Mutex
    for i := 0; i < 1000; i++ {
        w.Add(1)        
        go increment(&w, &m)
    }
    w.Wait()
    fmt.Println("final value of x", x)
}
```
[Run in playgroud](https://play.golang.org/p/OVdK0c-KfAj)

`Mutex`是一种结构类型，我们在第一行中初始化了一个`Mutex`类型的变量`m`。 在上面的程序中，我们修改了`increment`函数，使`x = x + 1`的代码在`m.Lock()`和`m.Unlock()`之间。现在这段代码没有任何竞态条件，因为在任何时候只允许一个`Goroutine`执行临界区。

现在如果运行该程序，它将输出，
```plain
final value of x 1000  
```

在第 18 行传递互斥锁的地址非常重要。如果通过值传递互斥锁而不是地址传递，则每个`Goroutine`都将拥有自己的互斥锁副本，那么肯定还会发生竞态条件。

## 使用通道`channel`解决竞态条件
我们也可以使用`channel`解决竞态条件。让我们看看这是如何完成的。
```golang
package main  
import (  
    "fmt"
    "sync"
    )
var x  = 0  
func increment(wg *sync.WaitGroup, ch chan bool) {  
    ch <- true
    x = x + 1
    <- ch
}
func main() {  
    var w sync.WaitGroup
    ch := make(chan bool, 1)
    for i := 0; i < 1000; i++ {
        w.Add(1)        
        go increment(&w, ch)
        wg.Done()
    }
    w.Wait()
    fmt.Println("final value of x", x)
}
```
[Run in playgroud](https://play.golang.org/p/XzS0VfM7N7E)

在上面的程序中，我们创建了一个容量为 1 的缓冲`channel`，并将其传递给第 18 行的`increment  Goroutine`。 此缓冲`channel`通过将`true`传递给`ch`来实现确保只有一个`Goroutine`访问临界区的。在`x`递增之前，由于缓冲`channel`的容量为 1，因此尝试写入此`channel`的所有其他`Goroutines`都会被阻塞，直到第 10 行将`ch`的值取出来。 使用这种方式，实现了只允许一个 Goroutine 访问临界区。

程序输出，
```plain
final value of x 1000  
```

## 互斥锁`Mutex` VS 通道`channel`
我们使用互斥锁`Mutex`和通道`channel`解决了竞态条件问题。那么我们怎么决定何时使用哪个？答案在于您要解决的问题。如果您尝试解决的问题更适合互斥锁`Mutex`，那么请继续使用`Mutex`。如果需要，请不要犹豫地使用`Mutex`。如果问题似乎更适合通道`channel`，那么使用它:)。

大多数 Go 新手尝试使用`channel`解决遇到的并发问题，因为它是该语言的一个很酷的功能。这是错的。该语言为我们提供了`Mutex`或`Channel`的选项，并且选择任何一种都没有错。

一般情况下，当`Goroutine`需要相互通信时使用`channel`，当`Goroutine`只访问代码的临界区时，使用`Mutex`。

在我们上面那些问题的情况下，我宁愿使用`Mutex`，因为这个问题不需要`goroutines`之间进行任何通信。因此`Mutex`是一种自然的选择。

我的建议是选择工具去解决问题，而不要为了工具去适应问题:)

