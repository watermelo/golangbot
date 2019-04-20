> * 原文地址：[Part 10: Switch Statement](https://golangbot.com/switch/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

`switch`是一个条件语句，它计算表达式并将其和可能的结果进行匹配，根据匹配执行代码块。它常用来替代多个 if else 子句。

光说不练假把式，让我们从一个简单的例子开始，该例子将手指号作为输入并输出该手指的名称:)。例如 1 是拇指，2 是无名指，依此类推。

```golang
package main

import (  
    "fmt"
)

func main() {  
    finger := 4
    switch finger {
    case 1:
        fmt.Println("Thumb")
    case 2:
        fmt.Println("Index")
    case 3:
        fmt.Println("Middle")
    case 4:
        fmt.Println("Ring")
    case 5:
        fmt.Println("Pinky")

    }
}
```
[Run in playground](https://play.golang.org/p/q4kjm2kpVe)

在上述程序中，`switch finger`将`finger`的值与每个 case 语句进行比较。该语句从上到下进行比较，并执行与表达式匹配的第一个`case`。在这个例子中，`finger`的值为 4，因此打印`Ring`。

具有相同值的`case`是不被允许的。如果你试图运行下面的程序，编译器会报错`main.go:18:2: duplicate case 4 in switch previous case at tmp/sandbox887814166/main.go:16:7`

```golang
package main

import (  
    "fmt"
)

func main() {  
    finger := 4
    switch finger {
    case 1:
        fmt.Println("Thumb")
    case 2:
        fmt.Println("Index")
    case 3:
        fmt.Println("Middle")
    case 4:
        fmt.Println("Ring")
    case 4://duplicate case
        fmt.Println("Another Ring")
    case 5:
        fmt.Println("Pinky")

    }
}
```
[Run in playground](https://play.golang.org/p/SfXdChWdoN)

## 默认的 case

我们手中只有 5 个手指。输入错误的手指号会发生什么？这是 默认的 `case`的用武之地。当其他`case`都不匹配时，将执行默认`case`。

```golang
package main

import (  
    "fmt"
)

func main() {  
    switch finger := 8; finger {
    case 1:
        fmt.Println("Thumb")
    case 2:
        fmt.Println("Index")
    case 3:
        fmt.Println("Middle")
    case 4:
        fmt.Println("Ring")
    case 5:
        fmt.Println("Pinky")
    default: //default case
        fmt.Println("incorrect finger number")
    }
}
```

[Run in playground](https://play.golang.org/p/Fq7U7SkHe1)

在上面的程序中，`finger`的值是 8 并且它与任何`case`都不匹配，因此打印`incorrect finger number`。默认`case`不一定是 switch 语句中的最后一个。它可以出现在`switch`的任何位置。

你可能注意到声明`finger`的一个小变化，它在`switch`中声明。`switch`可以包含一个可选的语句，它在表达式计算之前执行。在该行，`switch finger := 8; finger`，首先声明了`finger`并在表达式中使用，而`finger`的范围仅限于`switch`代码块。

## case 可以拥有多个表达式
可以在一个`case`中用逗号分隔实现多个表达式。

```golang
package main

import (  
    "fmt"
)

func main() {  
    letter := "i"
    switch letter {
    case "a", "e", "i", "o", "u": //multiple expressions in case
        fmt.Println("vowel")
    default:
        fmt.Println("not a vowel")
    }
}
```
[Run in playground](https://play.golang.org/p/Zs9Ek5SInh)

上面的程序检查字母是否是元音。`case "a", "e", "i", "o", "u":`匹配所有元音。由于`letter`的值为`i`该程序输出`vowel`

## 表达式 switch

`switch`中的表达式是可选的，可以省略。如果省略表达式，则`switch true`，每个`case`表达式都视为`true`，并且执行相应的代码块。

```golang
package main

import (  
    "fmt"
)

func main() {  
    num := 75
    switch { // expression is omitted
    case num >= 0 && num <= 50:
        fmt.Println("num is greater than 0 and less than 50")
    case num >= 51 && num <= 100:
        fmt.Println("num is greater than 51 and less than 100")
    case num >= 101:
        fmt.Println("num is greater than 100")
    }

}
```
[Run in playground](https://play.golang.org/p/mMJ8EryKbN)

在上述程序中，`switch`没有表达式，所以它被认为是`true`，并且会执行每一个`case`。`case num >= 51 && num <= 100:`为`true`且程序输出`num is greater than 51 and less than 100`。这种类型的`switch`可以用来替代多个`if else`的情况。

## fallthrough
在 Go 语言中，在执行`case`后立即从`switch`语句中返回。 `fallthrough`语句用于执行该`case`之后，再执行下一个`case`语句。

```golang
package main

import (
	"fmt"
)

func number() int {
	num := 15 * 5
	return num
}

func main() {

	switch num := number(); { //num is not a constant
	case num < 50:
		fmt.Printf("%d is lesser than 50\n", num)
		fallthrough
	case num < 100:
		fmt.Printf("%d is lesser than 100\n", num)
		fallthrough
	case num < 200:
		fmt.Printf("%d is lesser than 200", num)
	}

}
```
[Run in playground](https://play.golang.org/p/svGJAiswQj)

`switch`和`case`表达式不一定只能是常量。它们也可以在运行时进行赋值。在上面的程序中，`num`被初始化为函数`number()`的返回值。然后进入`case`语句的计算，`case num < 100:`为`true`则程序打印`75 is lesser than 100`，下一个语句是`fallthrough`。当遇到`fallthrough`时，将会执行下一个`case`的语句，并且打印`75 is lesser than 200`，程序的输出是

```plain
75 is lesser than 100  
75 is lesser than 200
```

`fallthrough`应该位于`case`语句中的最后。如果它出现在中间的某个位置，编译器会抛出错误`fallthrough statement out of place`

