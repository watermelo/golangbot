> * 原文地址：[Part 18: Interfaces - I](https://golangbot.com/interfaces-part-1/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

## 什么是接口
面向对象世界中接口的定义是“接口定义对象的行为”。它只指定对象应该做什么。实现此行为（实现细节）的方法取决于对象。

在 Go 的世界里，接口是一组方法签名。当一个类型为接口中的所有方法提供定义时，就说它实现了该接口。它与 OOP 世界非常相似。接口指定类型应具有的方法，类型决定如何实现这些方法。

例如，`WashingMachine`是具有方法签名`Cleaning()`和`Drying()`的接口。任何提供`Cleaning()`和`Drying()`定义的类型都可以说是实现了`WashingMachine`接口。

## 声明和实现接口
让我们来实现一个接口。
```golang
package main

import (  
    "fmt"
)

//interface definition
type VowelsFinder interface {  
    FindVowels() []rune
}

type MyString string

//MyString implements VowelsFinder
func (ms MyString) FindVowels() []rune {  
    var vowels []rune
    for _, rune := range ms {
        if rune == 'a' || rune == 'e' || rune == 'i' || rune == 'o' || rune == 'u' {
            vowels = append(vowels, rune)
        }
    }
    return vowels
}

func main() {  
    name := MyString("Sam Anderson")
    var v VowelsFinder
    v = name // possible since MyString implements VowelsFinder
    fmt.Printf("Vowels are %c", v.FindVowels())

}
```
[Run in playgroud](https://play.golang.org/p/F-T3S_wNNB)

上述程序的第 8 行创建了一个名为`VowelsFinder`的接口类型，它有一个方法`FindVowels() []rune`。

下一行创建了`MyString`类型。

在第 15 行，我们将`FindVowels() []rune`方法添加到接收者类型`MyString`中。现在`MyString`实现了`VowelsFinder`接口。这与 Java 等其他语言完全不同，其中类必须明确声明使用`implements`关键字去实现接口，而在 go 中这是是不需要的，如果类型包含接口中声明的所有方法，那么就说 go 实现了接口。

在第 28 行中，我们将类型为`MyString`的`name`赋值给`VowelsFinder`类型的`v`。因为`MyString`实现了`VowelsFinder`方法，所以这是可行的。 `v.FindVowels()`在下一行调用`MyString`类型的`FindVowels`方法并打印字符串 Sam Anderson 中的所有元音。这个程序输出`Vowels are [a e o]`

恭喜！你已创建并实现了你的第一个接口。

## 接口的使用

上面的例子告诉我们如何创建和实现接口，但它并没有真正展示接口的实际用途。如果我们在上面的程序中使用`name.FindVowels()`而不是`v.FindVowels()`，它也会工作，并且不会使用创建的接口。

现在让我们来看一下接口的实际用例。

我们将编写一个简单的程序，根据员工的个人工资计算公司的总支出。为简洁起见，我们假设所有费用均以美元计算。

```golang
package main

import (  
    "fmt"
)

type SalaryCalculator interface {  
    CalculateSalary() int
}

type Permanent struct {  
    empId    int
    basicpay int
    pf       int
}

type Contract struct {  
    empId  int
    basicpay int
}

//salary of permanent employee is sum of basic pay and pf
func (p Permanent) CalculateSalary() int {  
    return p.basicpay + p.pf
}

//salary of contract employee is the basic pay alone
func (c Contract) CalculateSalary() int {  
    return c.basicpay
}

/*
total expense is calculated by iterating though the SalaryCalculator slice and summing  
the salaries of the individual employees  
*/
func totalExpense(s []SalaryCalculator) {  
    expense := 0
    for _, v := range s {
        expense = expense + v.CalculateSalary()
    }
    fmt.Printf("Total Expense Per Month $%d", expense)
}

func main() {  
    pemp1 := Permanent{1, 5000, 20}
    pemp2 := Permanent{2, 6000, 30}
    cemp1 := Contract{3, 3000}
    employees := []SalaryCalculator{pemp1, pemp2, cemp1}
    totalExpense(employees)

}
```
[Run in playground](https://play.golang.org/p/5t6GgQ2TSU)

上面程序的第 7 行个用单个方法`CalculateSalary() int`声明了`SalaryCalculator`接口类型。

我们公司有两种员工，永久`Permanent`和合同`Contract`由第一行的结构定义。`Permanent`雇员的工资是基本工资和工资的总和，而对于`Contract`雇员来说，只需要基本的工资支付，分别在 23 和 28 行相应的`CalculateSalary`方法中表示。通过声明此方法，`Permanent`和`Contract`现在都实现了`SalaryCalculator`接口。

第 36 行中声明的`totalExpense`函数体现了使用接口的美妙之处。此方法使用`SalaryCalculator`接口的切片`[] SalaryCalculator`作为参数。在第 49 行中，我们将一个包含`Permanent`和`Contract`类型的切片传递给`totalExpense`函数。在第 39 行，`totalExpense`函数通过调用相应类型的`CalculateSalary`方法来计算费用。

这样做的最大好处是`totalExpense`可以扩展到任何新员工类型，而无需更改任何代码。假如 说公司增加了一种具有不同薪资结构的新型员工`Freelancer`自由职业者。这个`Freelancer`可以在 slice 参数中传递给`totalExpense`，`totalExpense`函数甚至没有一行代码更改。这个方法将做它应该做的事情，而`Freelancer`也将实现`SalaryCalculator`接口:)。

程序输出，`Total Expense Per Month $14050`

## 接口的内部表示

接口在内部可以被认为是由元组`(type, value)`表示。 `type`是接口的基础具体类型，`value`保存具体类型的值。

让我们写一段代码加深理解，
```golang
package main

import (  
    "fmt"
)

type Tester interface {  
    Test()
}

type MyFloat float64

func (m MyFloat) Test() {  
    fmt.Println(m)
}

func describe(t Tester) {  
    fmt.Printf("Interface type %T value %v\n", t, t)
}

func main() {  
    var t Tester
    f := MyFloat(89.7)
    t = f
    describe(t)
    t.Test()
}
```
[Run in playground](https://play.golang.org/p/ZpQhhhs2920)

`Tester`接口有一个方法`Test()`，`MyFloat`类型实现该接口。在 24 行中，我们将`MyFloat`类型的变量`f`赋给 Tester 类型的`t`。现在`t`的具体类型是`MyFloat`，`t`的值是 89.7。第 17 行中的`describe`函数打印接口的值和具体类型。该程序输出
```plain
Interface type main.MyFloat value 89.7  
89.7  
```

## 空接口

没有方法的接口称为空接口。它用`interface{}`表示。由于空接口没有方法，因此所有类型都实现了空接口。

```golang
package main

import (  
    "fmt"
)

func describe(i interface{}) {  
    fmt.Printf("Type = %T, value = %v\n", i, i)
}

func main() {  
    s := "Hello World"
    describe(s)
    i := 55
    describe(i)
    strt := struct {
        name string
    }{
        name: "Naveen R",
    }
    describe(strt)
}
```
[Run in playground](https://play.golang.org/p/Fm5KescoJb)

在上面程序的第 7 行中，`describe(i interface{})`函数将空接口作为参数，因此可以传递任何类型。

我们将`string`，`int`和`struct`分别为 13，15 和 21 行传递给`describe`函数，这个程序打印，
```plain
Type = string, value = Hello World  
Type = int, value = 55  
Type = struct { name string }, value = {Naveen R}  
```

## 类型断言

类型断言用于获取接口的基础值。

`i.(T)`是用于获取具体类型为`T`的`i`接口的基础值的语法。

代码胜过言语。让我们写一个类型断言。

```golang
package main

import (  
    "fmt"
)

func assert(i interface{}) {  
    s := i.(int) //get the underlying int value from i
    fmt.Println(s)
}
func main() {  
    var s interface{} = 56
    assert(s)
}
```
[Run in playground](https://play.golang.org/p/YstKXEeSBL)

在第 12 行，`s`的具体类型是`int`。我们在第 8 行中使用语法`i.(int)`去获取`i`的`int`型基础值。该程序打印 56。

如果上述程序中的具体类型不是`int`，会发生什么？好吧，让我们找出来。
```golang
package main

import (  
    "fmt"
)

func assert(i interface{}) {  
    s := i.(int) 
    fmt.Println(s)
}
func main() {  
    var s interface{} = "Steven Paul"
    assert(s)
}

```
[Run in playground](https://play.golang.org/p/88KflSceHK)

在上面的程序中，我们将具体类型为`string`的`s`传递给`assert`函数，该函数尝试从中提取`int`值。该程序将产生 panic 内容 `panic: interface conversion: interface {} is string, not int.`

要解决上面的问题，我们只要使用下面的语法，
```plain
v, ok := i.(T)  
```

如果`i`的具体类型是`T`，则`v`将具有`i`的基础值，`ok`将为`true`。

如果`i`的具体类型不是`T`，那么`ok`将为`false`并且`v`将具有类型`T`的零值并且程序将不会发生混乱。

```golang
package main

import (  
    "fmt"
)

func assert(i interface{}) {  
    v, ok := i.(int)
    fmt.Println(v, ok)
}
func main() {  
    var s interface{} = 56
    assert(s)
    var i interface{} = "Steven Paul"
    assert(i)
}
```
[Run in playground](https://play.golang.org/p/0sB-KlVw8A)

当 Steven Paul 传递给`assert`函数时，`ok`将为`false`，因为`i`的具体类型不是`int`，而`v`将具有值`0`，即 int 的零值。该程序将打印，
```plain
56 true  
0 false  
```

## 类型 Switch

类型`switch`用于将接口的具体类型与各种`case`语句中指定的多种类型进行比较。它类似于`switch case`。唯一的区别是`case`指定类型而不是正常`switch`中的值。

`type switch`的语法类似于`Type`断言。将类型断言的语法`i.(T)`的类型`T`替换为类型`switch`的关键字`type`就行了。让我们看看下面的程序如何工作。

```golang
package main

import (  
    "fmt"
)

func findType(i interface{}) {  
    switch i.(type) {
    case string:
        fmt.Printf("I am a string and my value is %s\n", i.(string))
    case int:
        fmt.Printf("I am an int and my value is %d\n", i.(int))
    default:
        fmt.Printf("Unknown type\n")
    }
}
func main() {  
    findType("Naveen")
    findType(77)
    findType(89.98)
}
```
[Run in playground](https://play.golang.org/p/XYPDwOvoCh)

在上述程序的第 8 行中，`switch  i.(type)`指定了类型`switch`。每个`case`语句都将`i`的具体类型与特定类型进行比较。如果匹配，则打印相应的语句。该程序输出，
```plain
I am a string and my value is Naveen  
I am an int and my value is 77  
Unknown type  
```

第 20 行的`89.98`是`float64`类型，它不匹配任何`case`，因此在最后一行打印未知类型。

还可以将类型与接口进行比较。如果我们有一个类型，并且该类型实现了一个接口，则可以将该类型与它实现的接口进行比较。

来写一段代码让理解更清晰，

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

func (p Person) Describe() {  
    fmt.Printf("%s is %d years old", p.name, p.age)
}

func findType(i interface{}) {  
    switch v := i.(type) {
    case Describer:
        v.Describe()
    default:
        fmt.Printf("unknown type\n")
    }
}

func main() {  
    findType("Naveen")
    p := Person{
        name: "Naveen R",
        age:  25,
    }
    findType(p)
}
```
[Run in playgroud](https://play.golang.org/p/o6aHzIz4wC)

在上面的程序中，`Person`结构实现了`Describer`接口。在第 19 行中的 case 语句，将`v`与`Describer`接口类型进行比较。 `p`实现了`Describer`，因此满足了这种情况，当程序运行`findType(p)`时，调用了`Describe()`方法。

打印，
```plain
unknown type  
Naveen R is 25 years old  
```

接口的第一部分就结束了。我们将在第二部分中继续讨论接口。