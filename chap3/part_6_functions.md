> * 原文地址：[Part 6: Functions](https://golangbot.com/functions/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

## 什么是函数
函数是一个执行指定任务的代码块。函数接收输入，对输入执行一些计算并生成输出。

## 函数的声明
在 go 中声明函数的一般语法是

```golang
func functionname(parametername type) returntype {  
 //function body
}
```

函数声明以`func`关键字开头，然后是函数名。入参在`(`和`)`之间，返回类型在最后。用`parametername type`语法指定参数名和类型。也可以指定任意数量的参数，如`(parameter1 type, parameter2 type)`。函数的代码块被包含在`{`和`}`之间。

参数和返回类型在函数中是可选的。因此，以下语法也是有效的函数声明。

```golang
func functionname() {  
}
```

## 示例函数
让我们编写一个函数，它将产品的价格和数量作为入参，并通过取两个值的乘积来计算价格并返回。

```golang
func calculateBill(price int, no int) int {  
    var totalPrice = price * no
    return totalPrice
}
```

上面的函数有两个输入参数`price`和`no` 都是 int 类型，它返回`price`和`no`的乘积`totalPrice`，该返回值也是 int 类型。

如果连续的参数属于同一类型，可以在最后一个变量时写入一次类型来避免每个变量都声明类型。我可以将`int int, no int`写成`price, no int`。因此，上述功能可以改写为，

```golang
func calculateBill(price, no int) int {  
    var totalPrice = price * no
    return totalPrice
}
```

现在我们准备好了一个函数，让我们从代码中的某个地方调用它。调用函数的语法是`functionname(parameters)`。可以使用下面的代码调用上述函数。

```golang
calculateBill(10, 5)  
```

下面是完整的程序，它调用上述函数并打印总价。

```golang
package main

import (  
    "fmt"
)

func calculateBill(price, no int) int {  
    var totalPrice = price * no
    return totalPrice
}
func main() {  
    price, no := 90, 6
    totalPrice := calculateBill(price, no)
    fmt.Println("Total price is", totalPrice)
}
```
[Run in playground](https://play.golang.org/p/YJlW3g-VZH)

上述程序将打印，

```plain
Total price is 540  
```

## 多个返回值
函数可以返回多个值。我们可以编写一个函数`rectProps`，它获取矩形的长度和宽度，并返回矩形的面积和周长。矩形的面积是长度和宽度的乘积，周长是长度和宽度之和的两倍。

```golang
package main

import (  
    "fmt"
)

func rectProps(length, width float64)(float64, float64) {  
    var area = length * width
    var perimeter = (length + width) * 2
    return area, perimeter
}

func main() {  
     area, perimeter := rectProps(10.8, 5.6)
    fmt.Printf("Area %f Perimeter %f", area, perimeter) 
}
```
[Run in playground](https://play.golang.org/p/qAftE_yke_)

如果函数返回多个值，应该在`(`和`)`之间指定它们。`func rectProps(length, width float64)(float64, float64)`有两个 float64 类型的参数`length`和`width`，并且还返回两个 float64 的值。上述程序输出

```plain
Area 60.480000 Perimeter 32.800000  
```

## 命名返回值
可以从函数中返回一个命名值。如果命名了返回值，则可以将其视为在函数的第一行中声明了该变量。

可以使用命名返回值重写上述`rectProps`函数

```golang
func rectProps(length, width float64)(area, perimeter float64) {  
    area = length * width
    perimeter = (length + width) * 2
    return //no explicit return value
}
```

`area`和`perimeter`是上述函数中的命名返回值。请注意，函数中的`return`语句没有显式返回任何值。由于`area`和`perimeter`在函数声明中指定了返回值，因此在遇到`return`语句时会自动从函数返回它们。

## 空白标识符
`_`被称为 Go 中的空白标识符。它可以用来代替任何类型的任何值。让我们看看这个空白标识符的用途是什么。

`rectProps`函数返回矩形的面积和周长。如果我们只需要面积而不想要周长怎么办？这就是`_`发挥作用的地方。

下面的程序仅使用`rectProps`函数返回的面积。

```golang
package main

import (  
    "fmt"
)

func rectProps(length, width float64) (float64, float64) {  
    var area = length * width
    var perimeter = (length + width) * 2
    return area, perimeter
}
func main() {  
    area, _ := rectProps(10.8, 5.6) // perimeter is discarded
    fmt.Printf("Area %f ", area)
}
```
[Run in playground](https://play.golang.org/p/IkugSH1jIt)

在第 13 行，我们只使用了面积并且使用`_`标识符丢弃了周长参数。









