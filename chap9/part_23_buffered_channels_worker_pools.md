> * 原文地址：[Part 23: Buffered Channels and Worker Pools](https://golangbot.com/buffered-channels-worker-pools/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

## 什么是缓冲```channel```

我们在上一个教程中讨论的所有```channel```基本上都是无缓冲的。正如我们在```channel```教程中详细讨论的那样，发送和接收到无缓冲的```channel```都是阻塞的。

可以使用缓冲区创建```channel```。仅当缓冲区已满时才会阻塞对缓冲```channel```的发送。类似地，仅当缓冲区为空时才阻塞从缓冲```channel```接收。

可以通过添加一个```capacity```参数传递给```make```函数来创建缓冲```channel```，该函数指定缓冲区的大小。
```golang
ch := make(chan type, capacity)
```

对于具有缓冲区的```channel```，上述语法中的容量应大于 0。默认情况下，无缓冲通道的容量为 0，因此在上一个教程中创建通道时省略了容量参数。

我们来创建一个缓冲```channel```，

```golang
package main

import (
    "fmt"
)


func main() {
    ch := make(chan string, 2)
    ch <- "naveen"
    ch <- "paul"
    fmt.Println(<- ch)
    fmt.Println(<- ch)
}
```
[Run in playgroud](http://play.flysnow.org/p/gjBXmxagASP)

在上面的程序中，第 9 行我们创建一个容量为 2 的缓冲```channel```。由于```channel```的容量为 2，因此可以将 2 个字符串写入而不会被阻塞。我们在第 10 和 11 行写入 2 个字符串，随后读取了写入的字符串并打印，
```plain
naveen
paul
```
## 另一个例子
让我们再看一个缓冲```channel```的例子，其中```channel```的值写入```Goroutine```并从```main Goroutine```读取。这个例子将帮助我们更好地理解何时写入缓冲的```channel```。
```golang
package main

import (
    "fmt"
    "time"
)

func write(ch chan int) {
    for i := 0; i < 5; i++ {
        ch <- i
        fmt.Println("successfully wrote", i, "to ch")
    }
    close(ch)
}
func main() {
    ch := make(chan int, 2)
    go write(ch)
    time.Sleep(2 * time.Second)
    for v := range ch {
        fmt.Println("read value", v,"from ch")
        time.Sleep(2 * time.Second)

    }
}
```
[Run in playgroud](https://play.golang.org/p/bKe5GdgMK9)

上面的程序中，在第 16 行创建了容量为 2 的缓冲```channel ch```。```main Goroutine```将```ch```传给```write Goroutine```，然后```main Goroutine```休眠 2 秒钟。在此期间，```write Goroutine```在运行。```write Goroutine```有一个 for 循环，它将 0 到 4 的数字循环写入```channel ch```。由于容量为 2，因此能够将值 0 和 1 写入，然后阻塞直到从`channel ch`读取至少一个值。所以这个程序会立即打印以下 2 行，
```plain
successfully wrote 0 to ch
successfully wrote 1 to ch
```
在打印上述两行之后，```write Goroutine```中的写入被阻塞，直到```channel ch```的数据被读取。由于```main Goroutine```会休眠 2 秒，因此程序在接下来的 2 秒内不会打印任何内容。当```main Goroutine```在被唤醒后，使用```for range```循环开始从```channel ch```读取并打印读取值，然后再次休眠 2 秒，此循环继续，直到 ch 关闭。因此程序将在 2 秒后打印以下行，
```plain
read value 0 from ch
successfully wrote 2 to ch
```
然后继续直到所有值都写入并在关闭 ```channel```。最终的输出是，
```plain
successfully wrote 0 to ch
successfully wrote 1 to ch
read value 0 from ch
successfully wrote 2 to ch
read value 1 from ch
successfully wrote 3 to ch
read value 2 from ch
successfully wrote 4 to ch
read value 3 from ch
read value 4 from ch
```
## 死锁
```golang
package main

import (
    "fmt"
)

func main() {
    ch := make(chan string, 2)
    ch <- "naveen"
    ch <- "paul"
    ch <- "steve"
    fmt.Println(<-ch)
    fmt.Println(<-ch)
}
```
[Run in playgroud](http://play.flysnow.org/p/OTklpSVa9RJ)
在上面的程序中，我们将 3 个字符串写入容量为 2 的缓冲```channel```。当第三个字符串写入的时候已超过其容量，因此写入操作被阻塞。现在必须等待其他```Goroutine```从```channel```读取数据才能继续写入，但在上述代码中并没有从该```channel```读取数据的```Goroutine```。因此会出现死锁，程序将在运行时打印以下内容，
```plain
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
    /tmp/sandbox274756028/main.go:11 +0x100
```
##  长度 VS 容量

容量是```channel```可以容纳的值的数量。这是我们使用```make```函数创建时指定的值。

长度是当前在```channel```中的元素数量。

一个程序会让理解变得简单😀

```golang
package main

import (
    "fmt"
)

func main() {
    ch := make(chan string, 3)
    ch <- "naveen"
    ch <- "paul"
    fmt.Println("capacity is", cap(ch))
    fmt.Println("length is", len(ch))
    fmt.Println("read value", <-ch)
    fmt.Println("new length is", len(ch))
}
```
[Run in playgroud](http://play.flysnow.org/p/pEZBLtHqQml)

在上面的程序中，创建的```channel```容量为 3，即它可以容纳 3 个字符串。然后我们分别写入 2 个字符串，现在该```channel```有 2 个字符串，因此其长度为 2。 我们从```channel```中读取一个字符串。现在```channel```只有一个字符串了，因此它的长度变为 1。这个程序将打印，
```plain
capacity is 3
length is 2
read value naveen
new length is 1
```

## WaitGroup

本教程的下一部分是关于```Worker Pools```。要了解工作池，我们首先需要了解```WaitGroup```，因为它将用于工作池的实现。

```WaitGroup```用于阻塞```main Goroutines```直到所有```Goroutines```完成执行。比如说我们有 3 个从```main Goroutine```生成的```Goroutines```需要并发执行。```main Goroutines```需要等待其他 3 个```Goroutines```完成才能终止，否则可能在```main Goroutines```终止时，其余的```Goroutines```还没能得当执行，这种场景下可以使用```WaitGroup```来完成。

停止理论上代码😀

```golang
package main

import (
    "fmt"
    "sync"
    "time"
)

func process(i int, wg *sync.WaitGroup) {
    fmt.Println("started Goroutine ", i)
    time.Sleep(2 * time.Second)
    fmt.Printf("Goroutine %d ended\n", i)
    wg.Done()
}

func main() {
    no := 3
    var wg sync.WaitGroup
    for i := 0; i < no; i++ {
        wg.Add(1)
        go process(i, &wg)
    }
    wg.Wait()
    fmt.Println("All go routines finished executing")
}
```
[Run in playgroud](https://play.golang.org/p/CZNtu8ktQh)

```WaitGroup```是一种结构类型，我们在第 18 行创建一个```WaitGroup```类型的空值变量。 ```WaitGroup```的工作方式是使用计数器。当我们在```WaitGroup```上调用```int```型参数调用```Add``` 方法，计数器会增加传递给```Add```的值。递减计数器的方法是在```WaitGroup上```调用```Done```方法。 ```Wait```方法阻塞调用它的```Goroutine```，直到计数器变为零。

在上面的程序中，我们在第 20 行调用```wg.Add(1)```循环迭代 3 次。所以计数器的值现在变成了 3。 for 循环也产生 3 个```Goroutines```，```main Goroutines```在第 23 行调用了```wg.Wait()```以阻塞直到计数器变为零。在```Goroutine```中，通过调用```wg.Done```来减少计数器的值。 一旦所有 3 个生成的```Goroutines```完成执行，也就是```wg.Done()```被调用三次，计数器被清零，```main Goroutine```被解除阻塞，程序执行完成，输出，
```plain
started Goroutine  2
started Goroutine  0
started Goroutine  1
Goroutine 0 ended
Goroutine 2 ended
Goroutine 1 ended
All go routines finished executing
```

大家的输出可能与我的不同，因为```Goroutines```的执行顺序会有所不同:)。

## 协程池的实现

缓冲```channel```的一个重要用途是[协程池](https://en.wikipedia.org/wiki/Thread_pool)的实现。

通常，协程池是一组协程，它们等待分配给它们任务。一旦完成分配的任务，他们就会再次等待下一个任务。

我们将使用缓冲```channel```实现协程池。我们的协程池将执行查找输入数字的数字之和的任务。例如，如果传递 234，则输出将为 9（9 = 2 + 3 + 4）。协程池的输入将是伪随机整数列表。

以下是我们协程池的核心功能
* 创建一个```Goroutines```池，用于监听缓冲```jobs channel```，等待任务分配
* 向```jobs channel```添加任务
* 任务完成后，将结果写入缓冲```results channel```
* 从```results channel```读取和打印结果

我们将逐步编写此程序，以便更容易理解。

第一步是创建表示任务和结果的结构。

```golang
type Job struct {
    id       int
    randomno int
}

type Result struct {
    job         Job
    sumofdigits int
}
```
每个```Job```结构都有一个```id```和```randomno```，用来计算各个数字的总和。

```Result```结构有一个```job```字段和```sumofdigits```字段，```sumofdigits```字段用来保存```job```各个数字之和的结果。

下一步是创建用于接收任务和存储结果的缓冲```channel```。

```golang
var jobs = make(chan Job, 10)
var results = make(chan Result, 10)
```

```worker Goroutines```在任务缓冲```channel```上侦听新任务。一旦任务完成，把结果写入结果缓冲```channel```。

```digits```函数执行查找整数的各个数字之和并返回它的。我们为此函数添加 2 秒的休眠，以模拟此函数计算结果需要一些时间的场景。
```golang
func digits(number int) int {
    sum := 0
    no := number
    for no != 0 {
        digit := no % 10
        sum += digit
        no /= 10
    }
    time.Sleep(2 * time.Second)
    return sum
}
```

接下来将编写一个创建```worker Goroutine```的函数。
```golang
func worker(wg *sync.WaitGroup) {
    for job := range jobs {
        output := Result{job, digits(job.randomno)}
        results <- output
    }
    wg.Done()
}
```
上面的函数创建了一个```worker```，它从```jobs channel```读取任务，使用当前任务和```digits```函数的返回值创建```Result```结构，然后将结果写入结果缓冲```channel```。此函数将```WaitGroup wg```作为参数，在所有任务完成后，它将调用```Done```方法结束当前```Goroutine```的阻塞。

```createWorkerPool```函数将创建一个```Goroutines```池。
```golang
func createWorkerPool(noOfWorkers int) {
    var wg sync.WaitGroup
    for i := 0; i < noOfWorkers; i++ {
        wg.Add(1)
        go worker(&wg)
    }
    wg.Wait()
    close(results)
}
```

上面的函数将要创建的```worker```数量作为参数。它在创建```Goroutine```之前调用了```wg.Add(1)```来增加```WaitGroup```计数器。然后它通过将```WaitGroup wg```的地址传递给```worker```函数来创建```worker Goroutines```。在创建了所需的```worker Goroutines```之后，它通过调用```wg.Wait()```来阻塞当前协程直到所有```Goroutines```完成执行后，关闭```results channel```，因为所有的```Goroutines```都已完成执行，没有结果被写入该```results channel```。

现在我们已经写好了协程池，让我们继续编写将任务分配给协程的功能。
```golang
func allocate(noOfJobs int) {
    for i := 0; i < noOfJobs; i++ {
        randomno := rand.Intn(999)
        job := Job{i, randomno}
        jobs <- job
    }
    close(jobs)
}
```
上面的```allocate```函数将要创建的任务数作为输入参数，生成最大值为 998 的伪随机数，使用随机数创建```Job```结构，并将 for 循环计数器的```i```作为```id```，然后将它们写入```jobs channel```。它在写完所有任务后关闭了```jobs channel```。

下一步是创建一个函数读取```results channel```并打印输出。

```golang
func result(done chan bool) {
    for result := range results {
        fmt.Printf("Job id %d, input random no %d , sum of digits %d\n", result.job.id, result.job.randomno, result.sumofdigits)
    }
    done <- true
}
```

```result```函数读取```results channel```并打印任务 ID，输入随机数和随机数的总和。```result```函数在打印所有结果后，将```true```写入```done channel```。

万事俱备，让我们把上面所有的功能用```main```函数串联起来。

```golang
func main() {
    startTime := time.Now()
    noOfJobs := 100
    go allocate(noOfJobs)
    done := make(chan bool)
    go result(done)
    noOfWorkers := 10
    createWorkerPool(noOfWorkers)
    <-done
    endTime := time.Now()
    diff := endTime.Sub(startTime)
    fmt.Println("total time taken ", diff.Seconds(), "seconds")
}
```

第 2 行我们首先将程序的执行开始时间存储起来，在最后一行（第 12 行），我们计算```endTime```和```startTime```之间的时间差，并显示程序的总运行时间。这是必要的，因为我们将通过改变```Goroutines```的数量来做一些基准测试。

```noOfJobs```设置为 100，然后调用```allocate```以将任务添加到```jobs channel```。

然后创建```done channel```并将其传递给```results channel```，以便它可以开始打印输出并在打印完所有内容后通知。

最后，通过调用```createWorkerPool```函数创建了一个 10 个```work Goroutines```的池，然后```main```阻塞直到```done channel```写入```true```值，最后打印所有结果。

下面是完整的代码。

```golang
package main

import (
    "fmt"
    "math/rand"
    "sync"
    "time"
)

type Job struct {
    id       int
    randomno int
}
type Result struct {
    job         Job
    sumofdigits int
}

var jobs = make(chan Job, 10)
var results = make(chan Result, 10)

func digits(number int) int {
    sum := 0
    no := number
    for no != 0 {
        digit := no % 10
        sum += digit
        no /= 10
    }
    time.Sleep(2 * time.Second)
    return sum
}
func worker(wg *sync.WaitGroup) {
    for job := range jobs {
        output := Result{job, digits(job.randomno)}
        results <- output
    }
    wg.Done()
}
func createWorkerPool(noOfWorkers int) {
    var wg sync.WaitGroup
    for i := 0; i < noOfWorkers; i++ {
        wg.Add(1)
        go worker(&wg)
    }
    wg.Wait()
    close(results)
}
func allocate(noOfJobs int) {
    for i := 0; i < noOfJobs; i++ {
        randomno := rand.Intn(999)
        job := Job{i, randomno}
        jobs <- job
    }
    close(jobs)
}
func result(done chan bool) {
    for result := range results {
        fmt.Printf("Job id %d, input random no %d , sum of digits %d\n", result.job.id, result.job.randomno, result.sumofdigits)
    }
    done <- true
}
func main() {
    startTime := time.Now()
    noOfJobs := 100
    go allocate(noOfJobs)
    done := make(chan bool)
    go result(done)
    noOfWorkers := 10
    createWorkerPool(noOfWorkers)
    <-done
    endTime := time.Now()
    diff := endTime.Sub(startTime)
    fmt.Println("total time taken ", diff.Seconds(), "seconds")
}
```
[Run in playgroud](https://golangbot.com/buffered-channels-worker-pools/)

请在本地计算机上运行此程序，以便计算的总时间更准确。

程序将打印，
```plain
Job id 1, input random no 636, sum of digits 15
Job id 0, input random no 878, sum of digits 23
Job id 9, input random no 150, sum of digits 6
...
total time taken  20.01081009 seconds
```

对应于 100 个任务，将打印总共 100 行，将在最后一行打印该程序运行所花费的总时间。您的输出将与我的不同，因为```Goroutines```可以按任何顺序运行，总时间也会因硬件而异。在我的情况下，程序完成大约需要 20 秒。

现在让我们将```main```函数中的```noOfWorkers```增加到 20。我们将```worker```的数量增加了一倍。由于```work Goroutines```已经增加，程序完成所需的总时间应该减少。在我的情况下，它变成 10.004364685 秒，程序打印，
```plain
...
total time taken  10.004364685 seconds
```
现在我们了解到了随着```work Goroutines```数量的增加，完成任务所需的总时间减少了。我把它留作练习，让你在主函数中使用不同的```noOfJobs```和```noOfWorkers```的值执行并分析结果。

