> * 原文地址：[Part 19: Interfaces - II](https://golangbot.com/interfaces-part-2/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

## 指针接收者的接口 VS 值接收者的接口

我们在上一篇文章中讨论的所有示例接口都是使用值接收者实现的。也可以使用指针接收者实现接口。在使用指针接收者实现接口时需要注意一些细微之处。让我们使用以下程序了解一下。

```golang
package main

import "fmt"

type Describer interface {  
    Describe()
}
type Person struct {  
    name string
    age  int
}

func (p Person) Describe() { //implemented using value receiver  
    fmt.Printf("%s is %d years old\n", p.name, p.age)
}

type Address struct {  
    state   string
    country string
}

func (a *Address) Describe() { //implemented using pointer receiver  
    fmt.Printf("State %s Country %s", a.state, a.country)
}

func main() {  
    var d1 Describer
    p1 := Person{"Sam", 25}
    d1 = p1
    d1.Describe()
    p2 := Person{"James", 32}
    d1 = &p2
    d1.Describe()

    var d2 Describer
    a := Address{"Washington", "USA"}

    /* compilation error if the following line is
       uncommented
       cannot use a (type Address) as type Describer
       in assignment: Address does not implement
       Describer (Describe method has pointer
       receiver)
    */
    //d2 = a

    d2 = &a //This works since Describer interface
    //is implemented by Address pointer in line 22
    d2.Describe()

}
```
[Run in playground](https://play.golang.org/p/IzspYiAQ82)

在上面的程序中的第 13 行，`Person`结构使用值接收者实现了`Describer`接口。 

正如我们之前已经学过和讨论的方法那样，带有值接收者的方法同时接受指针和值接收者。用值或者值的解引用去调用值方法是合法的。

`p1`是`Person`类型的值，它在第 29 行中赋值给`d1`。`Person`实现了`d1`接口，因此 30 行，将打印`Sam is 25 years old.`

类似地，在第 32 行中将`&p2`赋值给`d1`。 因此第 33 行将打印`James is 32 years old.`，太棒了:)。

在第 22 行，`Address`结构使用指针接收者实现`Describer`接口。
如果上面程序的第 45 行没有被取消注释，我们将看到编译错误`main.go:42: cannot use a (type Address) as type Describer in assignment: Address does not implement Describer (Describe method has pointer receiver)`。这是因为，`Describer`接口是使用第地址指针接收者实现的，我们尝试分配一个值类型`a`，但它没有实现`Describer`接口。这肯定会让你感到惊讶，因为我们之前已经知道带有指针接收者的方法将同时接受指针和值接收者。那么第 45 行的代码为什么不行呢?

原因是在任何已经是指针或可以寻址的任何类型上调用指针值方法是合法的。而存储在接口中的具体值是不可寻址的，因此编译器不可能自动获取第 45 行`a`的地址，因此这段代码失败了。

第 47 行是正确的，因为我们将`a`的地址`&a`赋值给了`d2`。

该程序的其余部分是通俗易懂的。该程序将打印，

```plain
Sam is 25 years old  
James is 32 years old  
State Washington Country USA  
```

## 实现多个接口

一个类型可以实现多个接口。让我们看看如何在以下程序中完成此操作。
```golang
package main

import (  
    "fmt"
)

type SalaryCalculator interface {  
    DisplaySalary()
}

type LeaveCalculator interface {  
    CalculateLeavesLeft() int
}

type Employee struct {  
    firstName string
    lastName string
    basicPay int
    pf int
    totalLeaves int
    leavesTaken int
}

func (e Employee) DisplaySalary() {  
    fmt.Printf("%s %s has salary $%d", e.firstName, e.lastName, (e.basicPay + e.pf))
}

func (e Employee) CalculateLeavesLeft() int {  
    return e.totalLeaves - e.leavesTaken
}

func main() {  
    e := Employee {
        firstName: "Naveen",
        lastName: "Ramanathan",
        basicPay: 5000,
        pf: 200,
        totalLeaves: 30,
        leavesTaken: 5,
    }
    var s SalaryCalculator = e
    s.DisplaySalary()
    var l LeaveCalculator = e
    fmt.Println("\nLeaves left =", l.CalculateLeavesLeft())
}
```
[Run in playground](https://play.golang.org/p/DJxS5zxBcV)

上面程序在第 7 行和第 11 行分别声明了两个接口`SalaryCalculator`和`LeaveCalculator`。

在第 15 行中定义的`Employee`结构，实现了`SalaryCalculator`接口的`DisplaySalary`方法和`LeaveCalculator`接口的`CalculateLeavesLeft`方法。现在，`Employee`实现了`SalaryCalculator`和`LeaveCalculator`接口。

在第 41 行，我们将`e`赋值给`SalaryCalculator`接口类型的变量。在第 43 行，我们将相同的变量`e`分配给`LeaveCalculator`接口类型的变量。这就使得`Employee`类型的变量`e`实现了`SalaryCalculator`和`LeaveCalculator`接口。

程序输出，

```plain
Naveen Ramanathan has salary $5200  
Leaves left = 25  
```

## 接口的嵌入

尽管 go 不提供继承，但可以通过嵌入其他接口来创建新接口。

我们来看看是怎么完成的。

```golang
package main

import (  
    "fmt"
)

type SalaryCalculator interface {  
    DisplaySalary()
}

type LeaveCalculator interface {  
    CalculateLeavesLeft() int
}

type EmployeeOperations interface {  
    SalaryCalculator
    LeaveCalculator
}

type Employee struct {  
    firstName string
    lastName string
    basicPay int
    pf int
    totalLeaves int
    leavesTaken int
}

func (e Employee) DisplaySalary() {  
    fmt.Printf("%s %s has salary $%d", e.firstName, e.lastName, (e.basicPay + e.pf))
}

func (e Employee) CalculateLeavesLeft() int {  
    return e.totalLeaves - e.leavesTaken
}

func main() {  
    e := Employee {
        firstName: "Naveen",
        lastName: "Ramanathan",
        basicPay: 5000,
        pf: 200,
        totalLeaves: 30,
        leavesTaken: 5,
    }
    var empOp EmployeeOperations = e
    empOp.DisplaySalary()
    fmt.Println("\nLeaves left =", empOp.CalculateLeavesLeft())
}
```
[Run in playground](https://play.golang.org/p/Hia7D-WbZp)

上面程序的第 15 行中的`EmployeeOperations`接口是通过嵌入`SalaryCalculator`和`LeaveCalculator`接口创建的。

如果一个类型提供了`SalaryCalculator`和`LeaveCalculator`接口中存在的方法的方法定义，就可以说实现了`EmployeeOperations`接口。

`Employee`结构实现了`EmployeeOperations`接口，因为它分别在第 29 行和第 33 行中的`DisplaySalary`和`CalculateLeavesLeft`方法提供了定义。

在第 46 行，类型为`Employee`的`e`被赋值给`EmployeeOperations`类型的`empOp`。在接下来的两行中，在`empOp`上调用`DisplaySalary()`和`CalculateLeavesLeft()`方法。

 程序输出，
```plain
Naveen Ramanathan has salary $5200  
Leaves left = 25  
```

## 接口的零值

接口的零值是`nil`。 `nil`接口的值和类型都为`nil`。

```golang
package main

import "fmt"

type Describer interface {  
    Describe()
}

func main() {  
    var d1 Describer
    if d1 == nil {
        fmt.Printf("d1 is nil and has type %T value %v\n", d1, d1)
    }
}
```

[Run in playground](https://play.golang.org/p/vwYHC6Y78H)

上述程序中的`d1`为`nil`，此程序将输出
```plain
d1 is nil and has type <nil> value <nil> 
```

如果我们尝试在`nil`接口上调用方法，程序将会发生`panic`，因为`nil`接口既没有具体值也没有具体类型。

```golang
package main

type Describer interface {  
    Describe()
}

func main() {  
    var d1 Describer
    d1.Describe()
}
```
[Run in playground](https://play.golang.org/p/rM-rY0uGTI)

由于上面程序中的`d1`是`nil`，因此程序将会出现`panic: runtime error: invalid memory address or nil pointer dereference 
[signal SIGSEGV: segmentation violation code=0xffffffff addr=0x0 pc=0xc8527]"`
