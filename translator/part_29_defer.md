> * 原文地址：[Part 29: Defer](https://golangbot.com/defer/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

## 什么是 Defer
在存在```defer```语句的函数返回之前，会执行```defer```的调用。定义可能看起来有点难懂，但通过示例来理解它非常简单。

```golang
package main

import (
    "fmt"
)

func finished() {
    fmt.Println("Finished finding largest")
}

func largest(nums []int) {
    defer finished()
    fmt.Println("Started finding largest")
    max := nums[0]
    for _, v := range nums {
        if v > max {
            max = v
        }
    }
    fmt.Println("Largest number in", nums, "is", max)
}

func main() {
    nums := []int{78, 109, 2, 563, 300}
    largest(nums)
}
```
[Run in playgroud](http://play.flysnow.org/p/3192v9QNJdP)
以上是一个简单的程序，用于查找给定切片最大的数。```largest```函数将```int```切片作为参数，并输出该切片的最大数。```largest```函数的第一行包含语句```defer finished()```。这意味着在```largest```函数返回之前将调用```finished```函数。运行此程序，可以看到以下输出。
```plain
Started finding largest
Largest number in [78 109 2 563 300] is 563
Finished finding largest
```
```largest```函数开始执行并打印上述输出的前两行。在它返回之前，```defer```函数完成执行，并打印```Finished finding largest``` :)

## ```defer```一个方法

```defer```不仅限于函数。```defer```调用方法也是完全合法的。让我们写一个小程序来测试它。
```golang
package main

import (
    "fmt"
)


type person struct {
    firstName string
    lastName string
}

func (p person) fullName() {
    fmt.Printf("%s %s",p.firstName,p.lastName)
}

func main() {
    p := person {
        firstName: "John",
        lastName: "Smith",
    }
    defer p.fullName()
    fmt.Printf("Welcome ")
}
```
[Run in playground](http://play.flysnow.org/p/GpDb1HPl5Kv)

在上面的程序中，我们```defer```了一个方法的调用，其余的代码是不难懂的。该程序输出，
```plain
Welcome John Smith
```

## ```defer```的参数作用域
```defer```的函数的参数是在执行```defer```语句时传入的，在实际函数调用的时候```defer```函数的参数还是当初传入的参数。

来一个例子，
```golang
package main

import (
    "fmt"
)

func printA(a int) {
    fmt.Println("value of a in deferred function", a)
}
func main() {
    a := 5
    defer printA(a)
    a = 10
    fmt.Println("value of a before deferred function call", a)

}
```
[Run in playgroud](http://play.flysnow.org/p/MUIHRyi-sbm)

在上面的程序中，第 11 行```a```被初始化为 5，```defer```语句实在第 12 行。```a```是```defer```函数```printA```的参数。在第 13 行我们将```a```的值更改为 10。该程序的输出，
```plain
value of a before deferred function call 10
value of a in deferred function 5
```
从上面的输出可以看到，尽管在执行```defer```语句之后```a```的值变为 10，但实际的```defer```函数调用```printA(a)```仍然打印 5。

## 多个```defer```函数的调用顺序
当一个函数有多个```defer```调用时，它们会被添加到栈中并以后进先出（LIFO）的顺序执行。
我们将编写一个小程序，使用一系列```defer```来反向打印字符串。
```golang
package main

import (
    "fmt"
)

func main() {
    name := "Naveen"
    fmt.Printf("Orignal String: %s\n", string(name))
    fmt.Printf("Reversed String: ")
    for _, v := range []rune(name) {
        defer fmt.Printf("%c", v)
    }
}
```
[Run in playgroud](http://play.flysnow.org/p/ZlABxLcjB3k)

在上面的程序中，第 11 行开始的```for range```循环迭代字符串并调用```defer fmt.Printf("%c", v)输出字符。这些```defer`调用将被添加到栈中并以后进先出的顺序执行，因此字符串将以相反的顺序打印。该程序将输出，
```plain
Orignal String: Naveen
Reversed String: neevaN
```
## ```defer```的实际用法
到目前为止我们看到的代码示例没有显示```defer```的实际用法。在本节中，我们将研究```defer```的一些实际用途。

```defer```用于应该执行函数调用的地方，而不管代码流程如何???。让我们用一个使用```WaitGroup```的例子来理解这一点。我们将首先编写程序而不使用```defer```，然后我们将修改它以使用```defer```，以此来理解```defer```是多么有用。
```golang
package main

import (
    "fmt"
    "sync"
)

type rect struct {
    length int
    width  int
}

func (r rect) area(wg *sync.WaitGroup) {
    if r.length < 0 {
        fmt.Printf("rect %v's length should be greater than zero\n", r)
        wg.Done()
        return
    }
    if r.width < 0 {
        fmt.Printf("rect %v's width should be greater than zero\n", r)
        wg.Done()
        return
    }
    area := r.length * r.width
    fmt.Printf("rect %v's area %d\n", r, area)
    wg.Done()
}

func main() {
    var wg sync.WaitGroup
    r1 := rect{-67, 89}
    r2 := rect{5, -67}
    r3 := rect{8, 9}
    rects := []rect{r1, r2, r3}
    for _, v := range rects {
        wg.Add(1)
        go v.area(&wg)
    }
    wg.Wait()
    fmt.Println("All go routines finished executing")
}
```
[Run in playground](http://play.flysnow.org/p/U0iRNwlD--C)
在上面的程序中，我们在第 8 行创建了一个```rect```结构，第 13 行给```rect```结构加上了```area```方法用于计算矩形的面积。此方法检查矩形的长度和宽度是否小于 0。如果是这样，它会打印相应的消息，否则会打印矩形的面积。

```main```函数创建了 3 个类型为```rect```的变量```r1```，```r2```和```r3```，将它们添加到```rects```切片中。然后使用```for range```循环迭代该切片，并将```area```方法并发执行。 ```WaitGroup wg```用于保证所有```Goroutines```执行完毕。```WaitGroup```作为参数传递给```area```方法，并在```area```方法中调用```wg.Done```，主要通知```main```该```Goroutine```已完成其工作。如果您仔细观察，可以看到这些调用恰好在```area```方法返回之前发生。无论代码采用哪个条件分支执行，都应在方法返回之前调用```wg.Done```，因此可以通过```defer```来解决这种场景。

来用```defer```重写上面的程序吧。

在下面的程序中，我们删除了上面程序中的 3 个```wg.Done```，并将其替换为```defer wg.Done()```，这将使代码更简洁易懂。
```golang
package main

import (
    "fmt"
    "sync"
)

type rect struct {
    length int
    width  int
}

func (r rect) area(wg *sync.WaitGroup) {
    defer wg.Done()
    if r.length < 0 {
        fmt.Printf("rect %v's length should be greater than zero\n", r)
        return
    }
    if r.width < 0 {
        fmt.Printf("rect %v's width should be greater than zero\n", r)
        return
    }
    area := r.length * r.width
    fmt.Printf("rect %v's area %d\n", r, area)
}

func main() {
    var wg sync.WaitGroup
    r1 := rect{-67, 89}
    r2 := rect{5, -67}
    r3 := rect{8, 9}
    rects := []rect{r1, r2, r3}
    for _, v := range rects {
        wg.Add(1)
        go v.area(&wg)
    }
    wg.Wait()
    fmt.Println("All go routines finished executing")
}
```
[Run in palyground](http://play.flysnow.org/p/L7kkorJy0Ez)
输出，
```plain
rect {8 9}'s area 72
rect {-67 89}'s length should be greater than zero
rect {5 -67}'s width should be greater than zero
All go routines finished executing
```

```defer```不仅能让程序简洁使用，在上述例子还有一个优点。假设我们使用新的```if```条件向```area```方法添加另一个处理分支。如果没有```defer```对```wg.Done```的调用，我们必须小心确保在这个新的处理分支中调用```wg.Done```。但由于对```wg.Done```的调用用了```defer```，我们再也不用担心这种情况了。相似的应用场景应该还有很多，比如打开文件的关闭等等。但是需要注意的是，大量的使用```defer```函数会导致程序运行效率变低。
