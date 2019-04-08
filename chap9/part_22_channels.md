> * 原文地址：[Part 22: Channels](https://golangbot.com/channels/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

在上一个教程中，我们讨论了 Go 中如何使用```Goroutines```实现并发。在本教程中，我们将讨论有关```channel```以及```Goroutines```如何使用```channel```进行通信。

## 什么是```channel```

```channel```可以被认为是```Goroutines```通信的管道。类似于水在管道中从一端流到另一端的方式，数据可以从一端发送，可以从另一端接收。

### ```channel```的声明
每个```channel```都有一个与之关联的类型。此类型是允许```channel```传输的数据类型。不允许使用该```channel```传输其他类型。

```chan T``` 代表类型为```T```的```channel```

```channel```的零值为```nil```。```nil channel```没有任何用处，因此得使用类似于```make map```和 ```make slice```来定义它。

让我们写一些声明```channel```的代码。

```golang
package main

import "fmt"

func main() {
    var a chan int
    if a == nil {
        fmt.Println("channel a is nil, going to define it")
        a = make(chan int)
        fmt.Printf("Type of a is %T", a)
    }
}
```
[Run in playground](http://play.flysnow.org/p/IhwRbqfpYqr)

在第 6 行声明了```var a chan int```，可以看到```channel```的零值为```nil```。因此，执行```if```条件内的语句并定义```channel```。上面的程序中的```a```是一个```int channel```。该程序将输出，
```plain
channel a is nil, going to define it
Type of a is chan int
```

使用```make```声明也是定义```channel```的有效而简洁的方法。
```golang
a := make(chan int)
```
上面的代码行定义了一个```int```型的```channel a```
## ```channel```的发送和接收
下面给出了从```channel```发送和接收数据的语法，
```plain
data := <- a // read from channel a
a <- data // write to channel a
```
箭头相对于通道的方向指定了是发送还是接收数据。

在第 1 行中，箭头从```a```向指向```data```，因此我们从通道```a```读取并将值存储到变量```data```中。

在第 2 行中，箭头指向```a```，因此我们把```data```写入通道```a```。

## 发送和接收默认是阻塞的

默认情况下，发送和接收是阻塞的。这是什么意思？当数据发送到```channel```时，发送方被阻塞直到其他```Goroutine```从该```channel```读取出数据。类似地，当从```channel```读取数据时，读取方被阻塞，直到其他```Goroutine```将数据写入该```channel```。

```channel```的这种属性有助于```Goroutines```有效地进行通信，而无需使用在其他编程语言中常见的显式锁或条件变量。

## ```channel```示例代码
让我们编写一个程序来了解```Goroutines```如何使用```channel```进行通信。

我们在上一篇教程中引用过这个程序。
```golang
package main

import (
    "fmt"
    "time"
)

func hello() {
    fmt.Println("Hello world goroutine")
}
func main() {
    go hello()
    time.Sleep(1 * time.Second)
    fmt.Println("main function")
}
```
[Run in playgroud](http://play.flysnow.org/p/4nT_Q6CuGrp)
这是上一个教程的代码，这里我们将使用```channel```重写上述程序。
```golang
package main

import (
    "fmt"
)

func hello(done chan bool) {
    fmt.Println("Hello world goroutine")
    done <- true
}
func main() {
    done := make(chan bool)
    go hello(done)
    <-done
    fmt.Println("main function")
}
```
[Run in playgroud](http://play.flysnow.org/p/mo-UkDH6Ygm)

在上面的程序中，我们在第一行创建了一个```bool```型的```done channel```。 并将其作为参数传递给```hello```。第 14 行我们正在从```done channel```接收数据。这行代码是阻塞的，这意味着在```Goroutine```将数据写入```done channel```之前将会一直阻塞。因此，上一个程序中的```time.Sleep```的就没有必要了，用```sleep```对程序而言是相当不友好。

代码行```<-done```表示从```done channel```接收数据，如果没有任何变量使用或存储该数据，这是完全合法的。

现在我们的```main Goroutine```被阻塞直到```done channel```有数据写入。 ```hello Goroutine```接收```done channel```作为参数，打印```Hello world goroutine```然后把```true```写入```done channel```。当这个写入完成时，```main Goroutine```从该```done channel```接收数据，然后结束阻塞打印了```main```函数的文本。

输出，
```plain
Hello world goroutine
main function
```

让我们通过在```hello Goroutine```中引入一个```sleep```来修改这个程序，以更好地理解这个阻塞概念。

```golang
package main

import (
    "fmt"
    "time"
)

func hello(done chan bool) {
    fmt.Println("hello go routine is going to sleep")
    time.Sleep(4 * time.Second)
    fmt.Println("hello go routine awake and going to write to done")
    done <- true
}
func main() {
    done := make(chan bool)
    fmt.Println("Main going to call hello go goroutine")
    go hello(done)
    <-done
    fmt.Println("Main received data")
}
```
[Run in playgroud](https://play.golang.org/p/EejiO-yjUQ)

这个程序将首先打印```Main going to call hello go goroutine```。然后```hello Goroutine```启动，打印```hello go routine is going to sleep```。打印完成后，```hello Goroutine```将休眠 4 秒钟，在此期间```main Goroutine```将被阻塞，因为它正在等待来自```<-done```的通道的数据。 4 秒后```hello Goroutine```苏醒，然后打印```hello go routine awake and going to write to done```并写入数据到```channel```，接着```main Goroutine```接收数据并打印```Main received data```。

## channel 的另外一个例子

让我们再写一个程序来更好地理解，该程序将打印数字各个位的平方和立方的总和。

例如，如果 123 是输入，则此程序将计算输出为

squares = (1 * 1) + (2 * 2) + (3 * 3)
cubes = (1 * 1 * 1) + (2 * 2 * 2) + (3 * 3 * 3)
output = squares + cubes = 50

我们将构建该程序，使得平方在一个```Goroutine```中计算，而立方在另一个```Goroutine```中进行计算，最终在```main Goroutine```中求和。

```golang
package main

import (
    "fmt"
)

func calcSquares(number int, squareop chan int) {
    sum := 0
    for number != 0 {
        digit := number % 10
        sum += digit * digit
        number /= 10
    }
    squareop <- sum
}

func calcCubes(number int, cubeop chan int) {
    sum := 0
    for number != 0 {
        digit := number % 10
        sum += digit * digit * digit
        number /= 10
    }
    cubeop <- sum
}

func main() {
    number := 589
    sqrch := make(chan int)
    cubech := make(chan int)
    go calcSquares(number, sqrch)
    go calcCubes(number, cubech)
    squares, cubes := <-sqrch, <-cubech
    fmt.Println("Final output", squares + cubes)
}
```
[Run in playgroud](http://play.flysnow.org/p/ijjNkGzHlfh)

```calcSquares```函数计算各个数字的平方的和，并将其发送到```squares channel```。类似地，```calcCubes```计算各个数字的立方的和并将其发送到```cubes channel```。

这两个函数都作为单独的```Goroutines```运行。每个函数都通过一个```channel```作为入参。```main Goroutine```等待来自这两个```channel```的数据。一旦从两个```channel```接收到数据，它们就存储在```squares```和`cubes`中求和，然后打印最终输出。该程序将打印，
```plain
Final output 1536
```

## 死锁

使用```channel```时要考虑的一个重要因素是死锁。如果```Goroutine```正在```channel```上发送数据，那么期待其他一些```Goroutine```接收数据。如果发送的数据没有被消费，程序将在运行时产生一个```panic```。

同样，如果```Goroutine```正在等待从一个```channel```接收数据，那么其他```Goroutine```应该在该```channel```上写入数据，否则程序也会出现```panic```。

```golang
package main


func main() {
    ch := make(chan int)
    ch <- 5
}
```
[Run in playgroud](http://play.flysnow.org/p/kCf9Ci4LeVK)

在上面的程序中，创建了一个```channel ch```，我们用```ch <-5```向```channel```发送 5。在该程序中，没有其他```Goroutine```从```ch```接收数据。因此，此程序将出现以下运行时错误。
```plain
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
    /tmp/sandbox249677995/main.go:6 +0x80
```
## 单向```channel```
到目前为止我们讨论的所有```channel```都是双向```channel```，即数据可以在它们上发送和接收。也可以创建单向```channel```，即仅发送或接收数据的```channel```。
```golang
package main

import "fmt"

func sendData(sendch chan<- int) {
    sendch <- 10
}

func main() {
    sendch := make(chan<- int)
    go sendData(sendch)
    fmt.Println(<-sendch)
}
```
[Run in playgroud](http://play.flysnow.org/p/pyWY9y3yDek)

在上面的程序中，我们在第 10 行中创建了仅发送```channel sendch```。```chan < -  int```表示当箭头指向```chan```时仅为发送```channel```。我们在第 12 行中尝试从该```channel```接收数据。 发现这是不允许的，当程序编译时，编译器会报错，
```plain
main.go:11: invalid operation: <-sendch (receive from send-only type chan<- int)
```

看起来好像没啥问题，但是一个写```channel```仅仅用来写，而不能用来读这样有啥意义!

我们接下来将用到```channel```转化。可以将双向```channel```转换为仅发送或仅接收的```channel```，反之亦然。

```golang
package main

import "fmt"

func sendData(sendch chan<- int) {
    sendch <- 10
}

func main() {
    chnl := make(chan int)
    go sendData(chnl)
    fmt.Println(<-chnl)
}
```
[Run in playgroud](http://play.flysnow.org/p/ux-QOQFL-qs)

在上面的程序第 10 行，创建了双向```channel chnl```。在第 11 行，它作为参数传递给```sendData Goroutine```，而```sendData```函数在第 5 行用```sendch chan < -  int```将此```chnl```转换为仅发送的```channel```类型。所以现在通道只在```sendData Goroutine```中是单向的，但它在```main Goroutine```中是双向的。该程序将打印 10 作为输出。(译者注：这就是单向```channel```的用途，定义函数或者方法的时候，使用只读或只写会让代码更健壮。)

## 关闭```channel```和循环```channel```

发送者能够关闭```channel```以通知接收者不再在该```channel```上发送数据。

接收者可以在从```channel```接收数据时使用额外的变量来检查```channel```是否已关闭。
```golang
v, ok := <- ch
```
在上面的语句中，如果成功地从该操作中接收到该值，则```ok```为```true```。如果```ok```为```false```，则表示我们正在从一个关闭的```channel```中读取。从关闭的```channel```中读取的值将是通道类型的零值。例如，如果是```int```类型，则从关闭的```channel```中读取到的值将为 0。
```golang
package main

import (
    "fmt"
)

func producer(chnl chan int) {
    for i := 0; i < 10; i++ {
        chnl <- i
    }
    close(chnl)
}
func main() {
    ch := make(chan int)
    go producer(ch)
    for {
        v, ok := <-ch
        if ok == false {
            break
        }
        fmt.Println("Received ", v, ok)
    }
}
```
[Run in playgroud](http://play.flysnow.org/p/w16yyF_ao98)

在上面的程序中，生产者```Goroutine```将 0 到 9 写入```channel chnl```，然后关闭它。在第 16 行```main```函数有一个无限```for```循环，它使变量```ok```检查```channel```是否被关闭。如果```ok```为```false```，则表示已关闭，因此循环中断。否则，将打印收到的值和```ok```的值。这个程序将打印，
```plain
Received  0 true
Received  1 true
Received  2 true
Received  3 true
Received  4 true
Received  5 true
Received  6 true
Received  7 true
Received  8 true
Received  9 true
```
for 循环的```for range```形式可用于从```channel```接收值，直到它被关闭。

让我们使用```for range```循环重写上面的程序。
```golang
package main

import (
    "fmt"
)

func producer(chnl chan int) {
    for i := 0; i < 10; i++ {
        chnl <- i
    }
    close(chnl)
}
func main() {
    ch := make(chan int)
    go producer(ch)
    for v := range ch {
        fmt.Println("Received ",v)
    }
}
```
[Run in playgroud](http://play.flysnow.org/p/glFvNvheZhW)

```for range```循环在第 16 行接收来自```channel ch```的数据直到它被关闭。 ```ch```关闭后，循环自动退出。该程序输出，
```plain
Received  0
Received  1
Received  2
Received  3
Received  4
Received  5
Received  6
Received  7
Received  8
Received  9
```
我们来重写一下上面那个求平方立方和的程序，

如果仔细查看程序，可以注意到在```calcSquares```函数和```calcCubes```函数中获取每一位的数字的逻辑重复了。我们将该逻辑的代码抽出来，然后分别在那两个函数中并发调用这个函数。

```golang
package main

import (
    "fmt"
)

func digits(number int, dchnl chan int) {
    for number != 0 {
        digit := number % 10
        dchnl <- digit
        number /= 10
    }
    close(dchnl)
}
func calcSquares(number int, squareop chan int) {
    sum := 0
    dch := make(chan int)
    go digits(number, dch)
    for digit := range dch {
        sum += digit * digit
    }
    squareop <- sum
}

func calcCubes(number int, cubeop chan int) {
    sum := 0
    dch := make(chan int)
    go digits(number, dch)
    for digit := range dch {
        sum += digit * digit * digit
    }
    cubeop <- sum
}

func main() {
    number := 589
    sqrch := make(chan int)
    cubech := make(chan int)
    go calcSquares(number, sqrch)
    go calcCubes(number, cubech)
    squares, cubes := <-sqrch, <-cubech
    fmt.Println("Final output", squares+cubes)
}
```
[Run in playgroud](http://play.flysnow.org/p/1gntVr-oZ_C)

上面程序中的```digits```函数现在包含从```number```中获取各位的逻辑，并且它同时由```calcSquares```和```calcCubes```函数调用。一旦```number```中没有更多的位，```channel```就会在第 13 行被关闭。 ```calcSquares```和```calcCubes Goroutines```使用```for range```循环监听各自的```channel```，直到它关闭。该程序的其余部分和之前的例子是相同的。该程序也会打印
```plain
Final output 1536
```

该节教程就结束了，```channel```中还有更多的概念，例如缓冲```channel```，```worker pool```和```select```。我们将在下一个教程中讨论它们。