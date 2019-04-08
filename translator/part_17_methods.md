> * 原文地址：[Part 17: Methods](https://golangbot.com/methods/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

## 什么是方法
方法是一个具有特殊接收者类型的函数，接收者在`func`关键字和方法名称之间。接收者可以是`struct`类型或非`struct`类型。接收者可用于方法内部的访问。

以下是创建方法的语法。
```golang
func (t Type) methodName(parameter list) {  
}
```
上面的代码片段创建了一个名为`methodName`的方法，该方法具有类型为`Type`的接收者。

## 方法的例子

让我们编写一个简单的程序，它在结构类型上创建一个方法并调用它。

```golang
package main

import (  
    "fmt"
)

type Employee struct {  
    name     string
    salary   int
    currency string
}

/*
 displaySalary() method has Employee as the receiver type
*/
func (e Employee) displaySalary() {  
    fmt.Printf("Salary of %s is %s%d", e.name, e.currency, e.salary)
}

func main() {  
    emp1 := Employee {
        name:     "Sam Adolf",
        salary:   5000,
        currency: "$",
    }
    emp1.displaySalary() //Calling displaySalary() method of Employee type
```
[Run in playgroud](https://play.golang.org/p/T4svXOTYSg)

在上面程序中的第 16 行，我们在`Employee`结构类型上创建了一个方法`displaySalary`。 `displaySalary()`方法可以访问其中的接收者`e Employee`。在第 17 行，我们使用接收者`e`并打印员工的姓名，币种和工资。

在第 26 行，我们使用语法`emp1.displaySalary()`调用了该方法，程序打印了，`Salary of Sam Adolf is $5000`

## 有了函数为啥还需要方法

我们仅使用函数来重写上面的程序。
```golang
package main

import (  
    "fmt"
)

type Employee struct {  
    name     string
    salary   int
    currency string
}

/*
 displaySalary() method converted to function with Employee as parameter
*/
func displaySalary(e Employee) {  
    fmt.Printf("Salary of %s is %s%d", e.name, e.currency, e.salary)
}

func main() {  
    emp1 := Employee{
        name:     "Sam Adolf",
        salary:   5000,
        currency: "$",
    }
    displaySalary(emp1)
}
```
[Run in playgroud](https://play.golang.org/p/dFwObgCUU0)

在上面的程序中，`displaySalary`从方法变为函数，`Employee`结构作为参数传递给它。这个程序也产生完全相同的输出`Salary of Sam Adolf is $5000`。

那么既然函数能实现一样的功能，为什么还需要方法呢。这有几个原因。让我们逐一看看它们。

- [Go 不是纯粹的面向对象编程语言](https://golang.org/doc/faq#Is_Go_an_object-oriented_language)，它不支持类。因此，类型上的方法是一种实现类似于类的行为的方法。
- 不同类型上可以定义具有相同名称的方法，而函数则不允许具有相同名称。让我们假设我们有一个`Square`和`Circle`结构。可以在`Square`和`Circle`上定义名为`Area`的方法。下面举个例子。
```golang
package main

import (  
    "fmt"
    "math"
)

type Rectangle struct {  
    length int
    width  int
}

type Circle struct {  
    radius float64
}

func (r Rectangle) Area() int {  
    return r.length * r.width
}

func (c Circle) Area() float64 {  
    return math.Pi * c.radius * c.radius
}

func main() {  
    r := Rectangle{
        length: 10,
        width:  5,
    }
    fmt.Printf("Area of rectangle %d\n", r.Area())
    c := Circle{
        radius: 12,
    }
    fmt.Printf("Area of circle %f", c.Area())
}
```
[Run in playgroud](https://play.golang.org/p/0hDM3E3LiP)

程序输出，
```plain
Area of rectangle 50  
Area of circle 452.389342  
```

方法的上述属性用到了接口的概念，我们将在下一个教程中讨论接口。

## 指针接收者 VS 值接收者

到目前为止，我们仅仅看到值接收者的方法。也可以使用指针接收者创建方法。值和指针接收者之间的区别在于，使用指针接收者的方法内部进行的更改对于调用者是可见的，而在值接收者中则不是这种情况。让我们在程序的帮助下理解这一点。
```golang
package main

import (  
    "fmt"
)

type Employee struct {  
    name string
    age  int
}

/*
Method with value receiver  
*/
func (e Employee) changeName(newName string) {  
    e.name = newName
}

/*
Method with pointer receiver  
*/
func (e *Employee) changeAge(newAge int) {  
    e.age = newAge
}

func main() {  
    e := Employee{
        name: "Mark Andrew",
        age:  50,
    }
    fmt.Printf("Employee name before change: %s", e.name)
    e.changeName("Michael Andrew")
    fmt.Printf("\nEmployee name after change: %s", e.name)

    fmt.Printf("\n\nEmployee age before change: %d", e.age)
    (&e).changeAge(51)
    fmt.Printf("\nEmployee age after change: %d", e.age)
}
```
[Run in playgroud](https://play.golang.org/p/tTO100HmUX)

在上面的程序中，`changeName`方法有一个值接收者`(e Employee)`，而`changeAge`方法有一个指针接收者`(e * Employee)`。对`changeName`中的`Employee`结构的名称字段所做的更改将对调用者不可见，因此程序在调用方法`e.changeName("Michael Andrew")`之前和之后打印相同的名称。由于`changeAge`方法使用了指针接收者`(e * Employee)`，因此调用方可以看到方法调用`(＆e).changeAge(51)`之后对`age`字段所做的更改。这个程序打印，
```plain
Employee name before change: Mark Andrew  
Employee name after change: Mark Andrew

Employee age before change: 50  
Employee age after change: 51  
```
在上面的程序的第 36 行，我们使用`(＆e).changeAge(51)`来调用`changeAge`方法。由于`changeAge`有一个指针接收者，我们使用了`(＆e)`来调用该方法。这不是必需的，语言为我们提供了使用`e.changeAge(51)`的选项。 在指针接收者的情况下，使用`e.changeAge(51)`将被语言解释为`(＆e).changeAge(51)`。

上述程序，用`e.changeAge(51)`替换`(＆e).changeAge(51)`也将输出一样的结果。
```golang
package main

import (  
    "fmt"
)

type Employee struct {  
    name string
    age  int
}

/*
Method with value receiver  
*/
func (e Employee) changeName(newName string) {  
    e.name = newName
}

/*
Method with pointer receiver  
*/
func (e *Employee) changeAge(newAge int) {  
    e.age = newAge
}

func main() {  
    e := Employee{
        name: "Mark Andrew",
        age:  50,
    }
    fmt.Printf("Employee name before change: %s", e.name)
    e.changeName("Michael Andrew")
    fmt.Printf("\nEmployee name after change: %s", e.name)

    fmt.Printf("\n\nEmployee age before change: %d", e.age)
    e.changeAge(51)
    fmt.Printf("\nEmployee age after change: %d", e.age)
}
```
[Run in playgroud](https://play.golang.org/p/nnXBsR3Uc8)

## 什么时候使用指针接收者&什么时候使用值接收者
通常，当调用者需要对方法所做的修改可见时，可以使用指针接收者。

指针接收者也可用于复制数据结构代价比较高的的地方。考虑一个包含许多字段的结构。使用此结构作为方法中的值接收者将需要复制整个结构，这代价是很高的。在这种情况下，如果使用指针接收者，则不会复制结构，并且只在该方法中使用指向它的指针。

在其他情况下，可以使用值接收者。

## 匿名字段的方法
可以调用属于结构的匿名字段的方法，就好像它们属于结构定义的一样。
```golang
package main

import (  
    "fmt"
)

type address struct {  
    city  string
    state string
}

func (a address) fullAddress() {  
    fmt.Printf("Full address: %s, %s", a.city, a.state)
}

type person struct {  
    firstName string
    lastName  string
    address
}

func main() {  
    p := person{
        firstName: "Elon",
        lastName:  "Musk",
        address: address {
            city:  "Los Angeles",
            state: "California",
        },
    }

    p.fullAddress() //accessing fullAddress method of address struct

}
```
[Run in playgroud](https://play.golang.org/p/vURnImw4_9)

在上面程序的第 32 行，我们使用`p.fullAddress()`调用`address`结构的`fullAddress()`方法。不需要用`p.address.fullAddress()`显式调用。这个程序打印
```plain
Full address: Los Angeles, California 
```

## 方法中的值接收者 VS 函数的值参数
大多数新手都有这个疑惑，我会尽量让它尽可能清楚😀。

当函数有一个值参数时，它只接受一个值参数。

当方法具有值接收者时，它将接受指针接受者和值接收者。

按惯例，上代码，
```golang
package main

import (  
    "fmt"
)

type rectangle struct {  
    length int
    width  int
}

func area(r rectangle) {  
    fmt.Printf("Area Function result: %d\n", (r.length * r.width))
}

func (r rectangle) area() {  
    fmt.Printf("Area Method result: %d\n", (r.length * r.width))
}

func main() {  
    r := rectangle{
        length: 10,
        width:  5,
    }
    area(r)
    r.area()

    p := &r
    /*
       compilation error, cannot use p (type *rectangle) as type rectangle 
       in argument to area  
    */
    //area(p)

    p.area()//calling value receiver with a pointer
}
```
[Run in playgroud](https://golangbot.com/methods/)

第 12 行中的函数`func area(r rectangle)`接受值参数，方法`func(r rectangle) area()`接受值接收者。

第 25 行，我们使用值参数`area(r)`调用 area 函数。类似地，我们使用值接收者调用 area 方法`r.area()`。

我们在第 28 行创建一个指针`p`指向`r`。在第 33 行，如果我们尝试将此指针传递给只接受值的函数 area，编译器将会报错，如果取消注释该行，则编译器将抛出编译错误`compilation error, cannot use p (type *rectangle) as type rectangle in argument to area`

现在是棘手的部分，在第 35 行中，代码`p.area()`中使用指针接收者`p`调用值接受者的方法 area，这完全有效。因为 area 有一个值接收者，为方便起见，Go 会把`p.area()`解析成`(* p).area()`。

程序会输出，
```plain
Area Function result: 50  
Area Method result: 50  
Area Method result: 50  
```

## 方法中的指针接收者 VS 函数的指针参数
与值参数类似，具有指针参数的函数将仅接受指针，而具有指针接收者的方法将接受值和指针接收者。

```golang
package main

import (  
    "fmt"
)

type rectangle struct {  
    length int
    width  int
}

func perimeter(r *rectangle) {  
    fmt.Println("perimeter function output:", 2*(r.length+r.width))

}

func (r *rectangle) perimeter() {  
    fmt.Println("perimeter method output:", 2*(r.length+r.width))
}

func main() {  
    r := rectangle{
        length: 10,
        width:  5,
    }
    p := &r //pointer to r
    perimeter(p)
    p.perimeter()

    /*
        cannot use r (type rectangle) as type *rectangle in argument to perimeter
    */
    //perimeter(r)

    r.perimeter()//calling pointer receiver with a value

}
```
[Run in playgroud](https://play.golang.org/p/Xy5wW9YZMJ)

在上述程序中的第 12 行，定义了一个函数`perimeter`，它接受一个指针参数，在第 17 行，定义了一种具有指针接收者的方法。

在第 27 行，我们用指针参数调用`perimeter`函数。在第 28 行，我们用指针接受者调用`perimeter`方法。

在注释行第 33 行中，我们尝试使用值参数`r`调用`perimeter`函数。这是不被允许的，因为带有指针参数的函数不接受值参数。如果取消该注释并且程序运行，编译将失败，错误为`main.go:33: cannot use r (type rectangle) as type *rectangle in argument to perimeter.`

在第 35 行中，我们使用值接收者`r`调用指针接收者的`perimeter`方法。这是允许的，为了方便，代码行`r.perimeter()`将被语言解释为`(＆r).perimeter()`。该程序将输出，
```plain
perimeter function output: 30  
perimeter method output: 30  
perimeter method output: 30  
```

## 非结构类型的方法
到目前为止，我们只在结构类型上定义了方法，也可以在非结构类型上定义方法。但是有一个需要注意，要在类型上定义方法，方法的接收者类型的定义和方法的定义应该在同一个包中。到目前为止，我们定义的结构上的所有结构和方法都位于同一包中，因此它们有效。
```golang
package main

func (a int) add(b int) {  
}

func main() {

}
```
[Run in playgroud](https://play.golang.org/p/ybXLf5o_lA)

在上面的程序中的第 3 行，我们试图在内置类型`int`中添加一个名为`add`的方法。这是不允许的，因为方法`add`的定义和`int`类型的定义不在同一个包中。这个程序会抛出编译错误`cannot define new methods on non-local type int`

让该段代码正确运行方法是为内置类型`int`创建类型别名，然后创建一个使用此类型别名作为接收者的方法。

```golang
package main

import "fmt"

type myInt int

func (a myInt) add(b myInt) myInt {  
    return a + b
}

func main() {  
    num1 := myInt(5)
    num2 := myInt(10)
    sum := num1.add(num2)
    fmt.Println("Sum is", sum)
}
```
[Run in playgroud](https://play.golang.org/p/sTe7i1qAng)

在上面程序的第 5 行中，我们为`int`创建了一个类型别名`myInt`。然后在第 7 行，我们定义了一个用`myInt`作为接收者的方法`add`。

程序将输出`Sum is 15.`
