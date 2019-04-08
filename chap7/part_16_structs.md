> * 原文地址：[Part 16: Structures](https://golangbot.com/structs/])
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

## 什么是结构体

结构是用户定义的类型，表示字段集合。它可以将一组数据放到结构中，而不是将它们作为单独的类型进行维护。

例如，员工具有`firstName`，`lastName`和`age`。将这三个属性组合成一个结构`employee`是有意义的。

## 声明一个结构
```golang
type Employee struct {  
    firstName string
    lastName  string
    age       int
}
```
上面的代码片段声明了一个结构类型`Employee`，其中包含`firstName`，`lastName`和`age`字段。通过在单行中声明属于同一类型的字段，后跟类型名称，可以使该结构更紧凑。在上面的结构体中，`firstName`和`lastName`属于同一个类型的字符串，因此可以将`struct`重写为
```golang
type Employee struct {  
    firstName, lastName string
    age, salary         int
}
```

上面的`Employee`结构称为命名结构，因为它创建了一个名为`Employee`的新类型。

也可以在不声明新类型的情况下声明结构，并且这些类型的结构称为匿名结构。

```golang
var employee struct {  
        firstName, lastName string
        age int
}
```

上面的代码片段创建了一个匿名结构`employee`。

## 创建一个命名结构
让我们使用一个简单的程序定义一个命名结构`Employee`。
```golang
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {

    //creating structure using field names
    emp1 := Employee{
        firstName: "Sam",
        age:       25,
        salary:    500,
        lastName:  "Anderson",
    }

    //creating structure without using field names
    emp2 := Employee{"Thomas", "Paul", 29, 800}

    fmt.Println("Employee 1", emp1)
    fmt.Println("Employee 2", emp2)
}
```
[Run in playgroud](https://play.golang.org/p/uhPAHeUwvK)

在上面程序的第 7 行中，我们创建了一个叫`Employee`的结构体。在第 15 行，初始化了一个`emp1`结构。字段名称的顺序不必与声明结构类型时的顺序相同。这里我们更改了`lastName`的位置并将其移到了最后。这将没有任何问题。

在第 23 行，通过省略字段名来初始化`emp2`。在这种情况下，必须保持字段的顺序与结构声明中指定的相同。
输出，
```plain
Employee 1 {Sam Anderson 25 500}  
Employee 2 {Thomas Paul 29 800} 
```

##  创建匿名结构体
```golang
package main

import (  
    "fmt"
)

func main() {  
    emp3 := struct {
        firstName, lastName string
        age, salary         int
    }{
        firstName: "Andreah",
        lastName:  "Nikola",
        age:       31,
        salary:    5000,
    }

    fmt.Println("Employee 3", emp3)
}
```
[Run in playgroud](https://play.golang.org/p/TEMFM3oZiq)

在上述程序的第 8 行，定义了匿名结构变量`emp3`。正如我们已经提到的，这被称为匿名结构，因为它只创建一个新的`struct`变量`emp3`，并没有定义任何新的结构类型。
该程序输出，
```plain
Employee 3 {Andreah Nikola 31 5000}  
```

## 结构体的零值
定义结构并且未使用任何值显式初始化它时，默认情况下会为结构的字段分配其零值。
```golang
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    var emp4 Employee //zero valued structure
    fmt.Println("Employee 4", emp4)
}
```
[Run in playgroud](https://play.golang.org/p/p7_OpVdFXJ)

上面的程序定义了`emp4`，但它没有用任何值初始化。因此，`firstName`和`lastName`被赋予字符串的零值，即""，而`age`，`salary`被赋予`int`的零值，即 0。此程序输出\
```plain
Employee 4 {  0 0}  
```
也可以为某些字段指定值而忽略其余字段。在这种情况下，忽略的字段将被赋予零值。
```golang
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    emp5 := Employee{
        firstName: "John",
        lastName:  "Paul",
    }
    fmt.Println("Employee 5", emp5)
}
```
[Run in playgroud](https://play.golang.org/p/w2gPoCnlZ1)
在上面的第 14 和 15 行，`firstName`和`lastName`被初始化，而`age`和`salary`则没有。因此，`age`和`salary`被赋予 0。该程序输出，
```plain
Employee 5 {John Paul 0 0}  
```

## 访问结构的字段
`.`操作符用来访问结构的字段
```golang
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    emp6 := Employee{"Sam", "Anderson", 55, 6000}
    fmt.Println("First Name:", emp6.firstName)
    fmt.Println("Last Name:", emp6.lastName)
    fmt.Println("Age:", emp6.age)
    fmt.Printf("Salary: $%d", emp6.salary)
}
```
[Run in playgroud](https://play.golang.org/p/GPd_sT85IS)

上面程序中的`emp6.firstName`访问`emp6`结构的`firstName`字段。该程序输出，
```plain
First Name: Sam  
Last Name: Anderson  
Age: 55  
Salary: $6000  
```

也可以创建零结构，然后稍后为其字段分配值。

```golang
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    var emp7 Employee
    emp7.firstName = "Jack"
    emp7.lastName = "Adams"
    fmt.Println("Employee 7:", emp7)
}
```
[Run in playgroud](https://play.golang.org/p/ZEOx10g7nN)

在上面的程序中，定义了`emp7`，然后为`firstName`和`lastName`赋值。该程序输出，
```plain
Employee 7: {Jack Adams 0 0}  
```

## 指向结构的指针
可以创建一个指向结构的指针。
```golang
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    emp8 := &Employee{"Sam", "Anderson", 55, 6000}
    fmt.Println("First Name:", (*emp8).firstName)
    fmt.Println("Age:", (*emp8).age)
}
```
[Run in playgroud](https://play.golang.org/p/ZEOx10g7nN)

上面程序中的`emp8`是指向`Employee`结构的指针。`(* emp8).firstName`是访问`firstName`字段的语法。该程序输出，
```plain
First Name: Sam  
Age: 55  
```
Go 语言可以使用`emp8.firstName`来访问`firstName`字段，从而可以不用通过显式解除引用`(* emp8).firstName`来访问。
```golang
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    emp8 := &Employee{"Sam", "Anderson", 55, 6000}
    fmt.Println("First Name:", emp8.firstName)
    fmt.Println("Age:", emp8.age)
}
```
[Run in playgroud](https://play.golang.org/p/0ZE265qQ1h)

我们使用`emp8.firstName`来访问上面程序中的`firstName`字段，这个程序同样输出，
```plain
First Name: Sam  
Age: 55  
```

## 匿名字段
可以使用包含不带字段名称去创建结构。这些字段称为匿名字段。

下面的代码片段创建了一个结构`Person`，它有两个匿名字段`string`和`int`
```golang
type Person struct {  
    string
    int
}
```
让我们使用匿名字段写一段代码，
```golang
package main

import (  
    "fmt"
)

type Person struct {  
    string
    int
}

func main() {  
    p := Person{"Naveen", 50}
    fmt.Println(p)
}
```
[Run in playgroud](https://play.golang.org/p/YF-DgdVSrC)

在上面的程序中，`Person`是一个带有两个匿名字段的结构。 `p := Person {“Naveen”，50}`定义了`Person`的变量。该程序输出`{Naveen 50}`。

匿名字段没有名称，默认情况下，匿名字段的名称也是其类型的名称。例如，在上面的`Person`结构的情况下，虽然字段是匿名的，但默认情况下它们采用字段类型的名称。所以`Person`结构有 2 个字段，名称为`string`和`int`。

```golang
package main

import (  
    "fmt"
)

type Person struct {  
    string
    int
}

func main() {  
    var p1 Person
    p1.string = "naveen"
    p1.int = 50
    fmt.Println(p1)
}
```
[Run in playgroud](https://play.golang.org/p/K-fGNxVyiA)

在上面程序的第 14 和 15 行中，我们使用它们的类型`string`和`int`作为字段名称来访问`Person`结构的匿名字段。上述程序的输出是，
```plain
{naveen 50}
```

## 嵌套结构
如果结构包含一个`struct`类型的字段那称该结构为嵌套结构。

```golang
package main

import (  
    "fmt"
)

type Address struct {  
    city, state string
}
type Person struct {  
    name string
    age int
    address Address
}

func main() {  
    var p Person
    p.name = "Naveen"
    p.age = 50
    p.address = Address {
        city: "Chicago",
        state: "Illinois",
    }
    fmt.Println("Name:", p.name)
    fmt.Println("Age:",p.age)
    fmt.Println("City:",p.address.city)
    fmt.Println("State:",p.address.state)
}
```
[Run in playgroud](https://play.golang.org/p/46jkQFdTPO)

上述程序中的`Person`结构有一个字段`address`，而该字段又是一个结构。将输出，
```plain
Name: Naveen  
Age: 50  
City: Chicago  
State: Illinois  
```

## 提升字段

一个结构中的匿名结构的字段称为提升字段，因为它们可以被访问就像它们属于原结构一样。这个定义有点复杂，所以让我们直接用

```golang
type Address struct {  
    city, state string
}
type Person struct {  
    name string
    age  int
    Address
}
```

在上面的代码片段中，`Person`结构有一个匿名字段`Address`，它是一个结构。现在，`Address`结构的字段即`city`和`state`被称为提升字段，因为它们可以和`Person`结构声明的字段一样被访问。
```golang
package main

import (  
    "fmt"
)

type Address struct {  
    city, state string
}
type Person struct {  
    name string
    age  int
    Address
}

func main() {  
    var p Person
    p.name = "Naveen"
    p.age = 50
    p.Address = Address{
        city:  "Chicago",
        state: "Illinois",
    }
    fmt.Println("Name:", p.name)
    fmt.Println("Age:", p.age)
    fmt.Println("City:", p.city) //city is promoted field
    fmt.Println("State:", p.state) //state is promoted field
}
```
[Run in playgroud](https://play.golang.org/p/OgeHCJYoEy)

在上述程序的第 26 和 27 行中，`p`访问提升字段`city`和`state`，就好像它们是在结构`p`本身中被声明了，然后使用语法`p.city`和`p.state`访问。该程序输出，
```plain
Name: Naveen  
Age: 50  
City: Chicago  
State: Illinois 
```

## 导出结构和字段

如果结构类型以大写字母开头，则它是导出类型，可以从其他包访问它。类似地，如果结构的字段以大写字母开头，则也可以从其他包中访问它们。

让我们编写一个包含自定义包的程序，以便更好地理解这一点。

在 go 工作区的 src 目录中创建一个名为 structs 的文件夹。在 structs 内创建另一个目录 computer。

进入 computer 目录，用 spec.go 为名称保存该代码。

```golang
package computer

type Spec struct { //exported struct  
    Maker string //exported field
    model string //unexported field
    Price int //exported field
}
```

上面的代码片段创建了一个包`computer`，其中包含一个导出的结构类型`Spec`，它包含两个导出字段`Maker`和`Price`以及一个非导出的字段`model`。让我们从`main package`导入这个包并使用`Spec`结构。

在 structs 目录中创建一个名为 main.go 的文件，并在 main.go 中编写以下程序
```golang
package main

import "structs/computer"  
import "fmt"

func main() {  
    var spec computer.Spec
    spec.Maker = "apple"
    spec.Price = 50000
    fmt.Println("Spec:", spec)
}
```
包结构应该如下所示，
```plain
src  
   structs
          computer
                  spec.go
          main.go
```

在上面程序的第 3 行中，我们导入了 computer 包。在第 8 和 9 行中，我们访问了`Spec`结构的两个导出字段`Maker`和`Price Spec`的价格。可以通过执行命令`go install structs`，然后再执行`workspacepath/bin/structs`来运行该程序

该程序将输出，`Spec: {apple  50000}`

如果我们尝试访问非导出的字段`model`，编译器会报错。我们用以下代码替换上述 main.go 的内容。
```golang
package main

import "structs/computer"  
import "fmt"

func main() {  
    var spec computer.Spec
    spec.Maker = "apple"
    spec.Price = 50000
    spec.model = "Mac Mini"
    fmt.Println("Spec:", spec)
}
```
在上述程序的第 10 行中，我们尝试访问非导出的字段`model`。运行此程序将导致编译错误`spec.model undefined (cannot refer to unexported field or method model)`

## 结构的相等性

结构是值类型，如果它们的每个字段都是可比较的，则可比较。两个结构变量的相应字段相等，则认为它们是相等的。

```golang
package main

import (  
    "fmt"
)

type name struct {  
    firstName string
    lastName string
}


func main() {  
    name1 := name{"Steve", "Jobs"}
    name2 := name{"Steve", "Jobs"}
    if name1 == name2 {
        fmt.Println("name1 and name2 are equal")
    } else {
        fmt.Println("name1 and name2 are not equal")
    }

    name3 := name{firstName:"Steve", lastName:"Jobs"}
    name4 := name{}
    name4.firstName = "Steve"
    if name3 == name4 {
        fmt.Println("name3 and name4 are equal")
    } else {
        fmt.Println("name3 and name4 are not equal")
    }
}
```
[Run in playgroud](https://play.golang.org/p/AU1FkdsPk7)
在上面的程序中，`name`结构包含两个字符串字段。由于字符串具有可比性，因此可以通过`name`结构的两个变量去比较。

在上面的程序中，name1 和 name2 相等，而 name3 和 name4 则不相等。该程序将输出，
```plain
name1 and name2 are equal  
name3 and name4 are not equal 
```

如果结构变量包含无法比较的字段，则该结构不具有可比性。

```golang
package main

import (  
    "fmt"
)

type image struct {  
    data map[int]int
}

func main() {  
    image1 := image{data: map[int]int{
        0: 155,
    }}
    image2 := image{data: map[int]int{
        0: 155,
    }}
    if image1 == image2 {
        fmt.Println("image1 and image2 are equal")
    }
}
```
[Run in playgroud](https://play.golang.org/p/T4svXOTYSg)

在上面的程序中，`image`结构类型包含一个`map`类型的字段`data`。由于`map`不具有可比性，因此无法比较`image1`和`image2`。如果运行此程序，编译将失败，错误为`main.go:18: invalid operation: image1 == image2 (struct containing map[int]int cannot be compared).`。








