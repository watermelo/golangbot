> * 原文地址：[Part 26: Structs Instead of Classes - OOP in Go](https://golangbot.com/structs-instead-of-classes/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

## Go 是面向对象的语言吗
Go 不是纯粹的面向对象的编程语言。摘自 Go 的常见问题解答，回答了 Go 是否面向对象的问题。

> 是也不是。尽管 Go 具有类型和方法，并且允许面向对象的编程风格，但是没有类型层次结构。 Go 中 “interface” 的概念提供了一种我们认为易于使用且在某些方面更为通用的方法。还有一些方法可以将类型嵌入到其他类型中，以提供类似但不完全相同的子类化。此外，Go 中的方法比 C ++或 Java 更通用：可以为任何类型的数据定义它们，甚至是内置类型，例如普通的整数。它们不限于结构（类）。

在即将到来的教程中，我们将讨论如何使用 Go 实现面向对象的编程概念。与其他面向对象的语言（如 Java）相比，它们中的一些在实现上有很大不同。

## 结构取代类
Go 没有提供类，但它提供了结构，可以在结构上添加方法。这提供了将数据和方法捆绑在一起的行为，类似于类。

让我们马上开始一个例子，以便更好地理解。

我们将在此示例中创建一个自定义包，因为它有助于更​​好地理解结构如何有效地替代类。

在 Go 工作区内创建一个文件夹，并将其命名为 oop。在 oop 中创建一个子文件夹 employee。在 employee 文件夹中，创建一个名为 employee.go 的文件

结构看起来像这样，

```plain
workspacepath -> oop -> employee -> employee.go
```

employee.go 的实际内容如下所示，
```golang
package employee

import (  
    "fmt"
)

type Employee struct {  
    FirstName   string
    LastName    string
    TotalLeaves int
    LeavesTaken int
}

func (e Employee) LeavesRemaining() {  
    fmt.Printf("%s %s has %d leaves remaining", e.FirstName, e.LastName, (e.TotalLeaves - e.LeavesTaken))
}
```
在上面的程序中，第一行指定此文件属于 employee 包。第 7 行声明了`Employee`结构，一个名为`LeavesRemaining`的方法被添加到`Employee`结构中，该方法计算并显示员工剩余的假期。现在我们有一个结构和一个方法，它运行在一个类似于类的结构上。

在 oop 文件夹下创建一个 main.go 文件。

现在文件的结构如下，
```plain
workspacepath -> oop -> employee -> employee.go  
workspacepath -> oop -> main.go  
```

main.go 的内容如下，
```golang
package main

import "oop/employee"

func main() {  
    e := employee.Employee {
        FirstName: "Sam",
        LastName: "Adolf",
        TotalLeaves: 30,
        LeavesTaken: 20,
    }
    e.LeavesRemaining()
}
```

我们在第 3 行导入了 `employee` 包，在 `main()` 函数中调用了 `Employee`结构的`LeavesRemaining()`方法。


此程序无法在 playground 上运行，因为它具有自定义包。如果你在本地运行这个程序，通过令`go install oop`后跟`workspacepath/bin/oop`，该程序将打印输出，
```plain
Sam Adolf has 10 leaves remaining  
```

## `New()` 函数取代构造函数
我们上面写的程序看起来不错，但它有一个小问题。让我们看看当我们定义零值的`Employee`结构时会发生什么。将 main.go 的内容更改为以下代码，
```golang
package main

import "oop/employee"

func main() {  
    var e employee.Employee
    e.LeavesRemaining()
}
```
我们做的唯一修改是在第 6 行创建了零值`Employee`。该程序将输出，

```plain
has 0 leaves remaining
```

如你所见，使用零值`Employee`的创建变量是不可用的。它没有有效的`FirstName`，`LastName`，也没有有效的休假详情。

在像 Java 这样的 OOP 语言中，这个问题可以通过构造函数来解决。可以使用参数化构造函数创建有效对象。

Go 不支持构造函数。如果类型的零值不可用，则需要程序员让类型不能导出使之不能从其他包访问，并且还要提供名为`NewT()`的函数，该函数使用所需的值初始化类型`T`。Go 中的一个约定是命名一个函数，它为`NewT()`创建一个 T 类型的值。这就像一个构造函数。如果包只定义了一种类型，那么 Go 中的一个约定就是将此函数命名为`New()`而不是`NewT()`。

让我们修改一下程序，以便每次创建员工时都可以使用。

第一步是让`Employee`结构不能被导出，并创建一个函数`New()`，它将创建一个新的`Employee`。将 employee.go 中的代码替换为以下代码，
```golang
package employee

import (  
    "fmt"
)

type employee struct {  
    firstName   string
    lastName    string
    totalLeaves int
    leavesTaken int
}

func New(firstName string, lastName string, totalLeave int, leavesTaken int) employee {  
    e := employee {firstName, lastName, totalLeave, leavesTaken}
    return e
}

func (e employee) LeavesRemaining() {  
    fmt.Printf("%s %s has %d leaves remaining", e.firstName, e.lastName, (e.totalLeaves - e.leavesTaken))
}
```
我们在这里做了一些重要的改变。我们将`Employee` 结构的起始字母`e`设为小写，即我们已将类型`Employee` 结构更改为类型`employee`结构。通过这样做，我们能让`employee`结构不能被导出，从而阻止了其他包的访问。除非需要指定导出它们，否则将不能导出的结构的所有字段都禁止导出是一种很好的做法。由于我们不需要在包外的任何地方使用`employee`结构的字段，因此我们也禁止了所有字段的导出。

我们在`LeavesRemaining()`方法中相应地修改了字段名。

现在，由于`employee`不能被导出，所以无法从其他包创建`Employee`类型的值。因此我们在第 14 行提供了一个导出的`New`函数，将所需参数作为输入并返回新创建的`employee`。

该程序仍然需要进行修改才能运行，但是让我们运行它来看一下修改的效果。如果运行此程序，它将失败并出现以下编译错误，
```plain
go/src/constructor/main.go:6: undefined: employee.Employee  
```

这是因为我们有一个不能被导出的`Employee`结构，因此编译器抛出错误，这个类型也没有在 main.go 中定义。这个错误正是我们想要的。现在没有其他包能够创建零值的`employee`。我们已成功阻止创建不可用的`employee`结构值。现在创建`employee`的唯一方法就是使用`New`函数。

用以下替换 main.go 的内容，
```golang
package main  

import "oop/employee"

func main() {  
    e := employee.New("Sam", "Adolf", 30, 20)
    e.LeavesRemaining()
}
```

对此文件的唯一修改是第 6 行，我们通过将所需参数传递给`New`函数来创建新`employee`。

以下是两个文件修改后的内容，

employee.go

```golang
package employee

import (  
    "fmt"
)

type employee struct {  
    firstName   string
    lastName    string
    totalLeaves int
    leavesTaken int
}

func New(firstName string, lastName string, totalLeave int, leavesTaken int) employee {  
    e := employee {firstName, lastName, totalLeave, leavesTaken}
    return e
}

func (e employee) LeavesRemaining() {  
    fmt.Printf("%s %s has %d leaves remaining", e.firstName, e.lastName, (e.totalLeaves - e.leavesTaken))
}
```

main.go

```golang
package main  

import "oop/employee"

func main() {  
    e := employee.New("Sam", "Adolf", 30, 20)
    e.LeavesRemaining()
}
```

程序输出，
```plain
Sam Adolf has 10 leaves remaining  
```

因此，你就能理解尽管 Go 不支持类，但可以使用结构替代类，使用`New()`方法去替代构造函数来有效地实现了类似的行为。
