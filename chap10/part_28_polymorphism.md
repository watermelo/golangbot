> * 原文地址：[Part 28: Polymorphism - OOP in Go
](https://golangbot.com/polymorphism/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

Go 中的多态性是在接口的帮助下实现的。正如我们已经讨论过的，接口可以在 Go 中隐式实现。如果类型为接口中声明的所有方法提供定义，则类型实现接口。让我们看看在接口的帮助下如何在 Go 中实现多态性。

## 使用接口实现多态
任何定义了接口的所有方法的类型都被称为隐式实现了该接口。

一个接口类型的变量可以承载任何实现该接口的值。接口的这个属性用于在 Go 中实现多态。

让我们在一个计算组织净收入的程序的帮助下来理解 Go 中的多态性。为简单起见，我们假设这个想象中的组织有两种项目的收入，即固定账单，时间和材料。该组织的净收入按这些项目的收入总和计算。为了简化本教程，我们假设货币是美元，我们不会处理美分。它将使用`int`表示。 （我建议阅读[该文章](https://forum.golangbridge.org/t/what-is-the-proper-golang-equivalent-to-decimal-when-dealing-with-money/413)以了解如何表示美分。感谢 Andreas Matuschek 在评论部分指出了这一点。）

我们定义一个`Income`结构，
```golang
type Income interface {  
    calculate() int
    source() string
}
```
上面定义的`Income`接口包含两个方法`calculate()`，它计算并返回项目来源收入的和，`source()`返回项目的名称。

下一步定义`FixedBilling`的结构
```golang
type FixedBilling struct {  
    projectName string
    biddedAmount int
}
```
`FixedBilling`项目有两个字段`projectName`，表示项目名称，`biddedAmount`是组织为项目出价的金额。

`TimeAndMaterial`结构表示按时间计算收益的项目
```golang
type TimeAndMaterial struct {  
    projectName string
    noOfHours  int
    hourlyRate int
}
```
`TimeAndMaterial`结构有三个字段`projectName`，`noOfHours`和`hourlyRate`。

下一步是定义这些结构类型的方法，这些方法计算并返回实际收入和收入来源。

```golang
func (fb FixedBilling) calculate() int {  
    return fb.biddedAmount
}

func (fb FixedBilling) source() string {  
    return fb.projectName
}

func (tm TimeAndMaterial) calculate() int {  
    return tm.noOfHours * tm.hourlyRate
}

func (tm TimeAndMaterial) source() string {  
    return tm.projectName
}
```
对于`FixedBilling`项目，收入只是项目的投标金额。因此我们从`FixedBilling`类型的`calculate()`方法返回它。

对于`TimeAndMaterial`项目，收入是`noOfHours`和`hourlyRate`的乘积。返回值来自于使用接收者类型为`TimeAndMaterial`的`calculate()`方法。

我们用`source()`方法来返回收入来源项目的名字。

由于`FixedBilling`和`TimeAndMaterial`结构都为`Income`接口的`calculate()`和`source()`方法提供了定义，因此两个结构都实现了`Income`接口。

```golang
func calculateNetIncome(ic []Income) {  
    var netincome int = 0
    for _, income := range ic {
        fmt.Printf("Income From %s = $%d\n", income.source(), income.calculate())
        netincome += income.calculate()
    }
    fmt.Printf("Net income of organisation = $%d", netincome)
}
```

上面的`calculateNetIncome`函数接受`Income`接口的切片作为参数。它通过迭代切片并在每个项目上调用`calculate()`方法来计算总收入。它还通过调用`source()`方法显示收入来源。根据`Income`接口的具体类型，将调用不同的`calculate()`和`source()`方法。因此，我们在`calculateNetIncome`函数中实现了多态性。

在将来，如果添加了一种新的收入来源，这个功能仍然可以正确计算总收入而无需一行代码更改:)。

接下来我们看看 main 函数的内容。

```golang
func main() {  
    project1 := FixedBilling{projectName: "Project 1", biddedAmount: 5000}
    project2 := FixedBilling{projectName: "Project 2", biddedAmount: 10000}
    project3 := TimeAndMaterial{projectName: "Project 3", noOfHours: 160, hourlyRate: 25}
    incomeStreams := []Income{project1, project2, project3}
    calculateNetIncome(incomeStreams)
}
```
在上面的 main 函数中，我们创建了三个项目，两个类型为`FixedBilling`，另一个类型为`TimeAndMaterial`。接下来，我们使用这 3 个项目创建一个类型为`Income`的切片。由于这些项目中的每一个都实现了`Income`接口，因此可以将所有三个项目添加到一个类型为`Income`的片段中。最后，我们用这个切片调用`calculateNetIncome`函数，它将显示各种收入和收入来源。

整个程序如下，
```golang
package main

import (  
    "fmt"
)

type Income interface {  
    calculate() int
    source() string
}

type FixedBilling struct {  
    projectName string
    biddedAmount int
}

type TimeAndMaterial struct {  
    projectName string
    noOfHours  int
    hourlyRate int
}

func (fb FixedBilling) calculate() int {  
    return fb.biddedAmount
}

func (fb FixedBilling) source() string {  
    return fb.projectName
}

func (tm TimeAndMaterial) calculate() int {  
    return tm.noOfHours * tm.hourlyRate
}

func (tm TimeAndMaterial) source() string {  
    return tm.projectName
}

func calculateNetIncome(ic []Income) {  
    var netincome int = 0
    for _, income := range ic {
        fmt.Printf("Income From %s = $%d\n", income.source(), income.calculate())
        netincome += income.calculate()
    }
    fmt.Printf("Net income of organisation = $%d", netincome)
}

func main() {  
    project1 := FixedBilling{projectName: "Project 1", biddedAmount: 5000}
    project2 := FixedBilling{projectName: "Project 2", biddedAmount: 10000}
    project3 := TimeAndMaterial{projectName: "Project 3", noOfHours: 160, hourlyRate: 25}
    incomeStreams := []Income{project1, project2, project3}
    calculateNetIncome(incomeStreams)
}
```
[Run in playground](https://play.golang.org/p/UClAagvLFT)

程序输出，
```plain
Income From Project 1 = $5000  
Income From Project 2 = $10000  
Income From Project 3 = $4000  
Net income of organisation = $19000 
```

## 在上述代码中添加新的收入来源

假设该组织通过做广告有了新的收入来源。让我们看看添加这个新的收入来源并计算总收入是多么简单，不需要对`calculateNetIncome`函数进行任何更改。由于多态性，这成为可能。

让我们首先定义一个`Advertisement`的收入类型，然后给该类型加上`calculate()`以及`source()`方法。

```golang
type Advertisement struct {  
    adName     string
    CPC        int
    noOfClicks int
}

func (a Advertisement) calculate() int {  
    return a.CPC * a.noOfClicks
}

func (a Advertisement) source() string {  
    return a.adName
}
```
`Advertisement`类型有三个字段：`adName`，`CPC`（每次点击费用）和`noOfClicks`（点击次数）。广告的总收入是`CPC`和`noOfClicks`的乘积。

让我们稍微修改一下 main 函数，以包含这个新的收入来源。

```golang
func main() {  
    project1 := FixedBilling{projectName: "Project 1", biddedAmount: 5000}
    project2 := FixedBilling{projectName: "Project 2", biddedAmount: 10000}
    project3 := TimeAndMaterial{projectName: "Project 3", noOfHours: 160, hourlyRate: 25}
    bannerAd := Advertisement{adName: "Banner Ad", CPC: 2, noOfClicks: 500}
    popupAd := Advertisement{adName: "Popup Ad", CPC: 5, noOfClicks: 750}
    incomeStreams := []Income{project1, project2, project3, bannerAd, popupAd}
    calculateNetIncome(incomeStreams)
}
```

我们创建了两个名为`bannerAd`和`popupAd`的广告收入。 `incomeStreams`切片包含我们刚刚创建的两个广告。

加上后的整个代码如下，
```golang
package main

import (  
    "fmt"
)

type Income interface {  
    calculate() int
    source() string
}

type FixedBilling struct {  
    projectName  string
    biddedAmount int
}

type TimeAndMaterial struct {  
    projectName string
    noOfHours   int
    hourlyRate  int
}

type Advertisement struct {  
    adName     string
    CPC        int
    noOfClicks int
}

func (fb FixedBilling) calculate() int {  
    return fb.biddedAmount
}

func (fb FixedBilling) source() string {  
    return fb.projectName
}

func (tm TimeAndMaterial) calculate() int {  
    return tm.noOfHours * tm.hourlyRate
}

func (tm TimeAndMaterial) source() string {  
    return tm.projectName
}

func (a Advertisement) calculate() int {  
    return a.CPC * a.noOfClicks
}

func (a Advertisement) source() string {  
    return a.adName
}
func calculateNetIncome(ic []Income) {  
    var netincome int = 0
    for _, income := range ic {
        fmt.Printf("Income From %s = $%d\n", income.source(), income.calculate())
        netincome += income.calculate()
    }
    fmt.Printf("Net income of organisation = $%d", netincome)
}

func main() {  
    project1 := FixedBilling{projectName: "Project 1", biddedAmount: 5000}
    project2 := FixedBilling{projectName: "Project 2", biddedAmount: 10000}
    project3 := TimeAndMaterial{projectName: "Project 3", noOfHours: 160, hourlyRate: 25}
    bannerAd := Advertisement{adName: "Banner Ad", CPC: 2, noOfClicks: 500}
    popupAd := Advertisement{adName: "Popup Ad", CPC: 5, noOfClicks: 750}
    incomeStreams := []Income{project1, project2, project3, bannerAd, popupAd}
    calculateNetIncome(incomeStreams)
}
```
[Run in playground](https://play.golang.org/p/BYRYGjSxFN)

上述程序输出，
```plain
Income From Project 1 = $5000  
Income From Project 2 = $10000  
Income From Project 3 = $4000  
Income From Banner Ad = $1000  
Income From Popup Ad = $3750  
Net income of organisation = $23750
```

您会注意到虽然我们添加了新的收入来源，但我们没有对`calculateNetIncome`函数进行任何更改。这就是多态性的作用。由于新的`Advertisement`类型也实现了`Income`接口，我们可以将它添加到`incomeStreams`切片中。 `calculateNetIncome`函数也没有任何变化，因为它能够调用`Advertisement`类型的`calculate()`和`source()`方法。




