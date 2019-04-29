> * 原文地址：[Part 8: if else statement](https://golangbot.com/if-statement/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

`if`是条件语句，语法为，

```golang
if condition {  
}
```
如果`condition`为`true`，介于`{}`之间的代码块将被执行。

与 C 之类的其他语言不同，即使`{}`之间只有一个语句，`{}`也是强制性需要的。

`else if`和`else`对于`if`来说是可选的。

```golang
if condition {  
} else if condition {
} else {
}
```

`if else`的数量不受限制，它们从上到下判断条件是否为真。如果`if else`或者`if`的条件为`true`，则执行相应的代码块。如果没有条件为真，则执行`else`的代码块。

让我们写一个简单的程序来查找数字是奇数还是偶数。

```golang
package main

import (  
    "fmt"
)

func main() {  
    num := 10
    if num % 2 == 0 { //checks if number is even
        fmt.Println("the number is even") 
    }  else {
        fmt.Println("the number is odd")
    }
}
```
[Run in playground](https://play.golang.org/p/vWfN8UqZUr)

`if num % 2 == 0`语句检查将数字除以 2 的结果是否为零。如果是，则打印`"the number is even"`，否则打印`"the number is odd"`。在上面的程序中，将打印`the number is even`。

`if`变量还可以包含一个可选的`statement`，它在条件判断之前执行。语法为

```golang
if statement; condition {  
}
```

让我们使用上面的语法重写程序，判断数字是偶数还是奇数。

```golang
package main

import (  
    "fmt"
)

func main() {  
    if num := 10; num % 2 == 0 { //checks if number is even
        fmt.Println(num,"is even") 
    }  else {
        fmt.Println(num,"is odd")
    }
}
```
[Run in playground](https://play.golang.org/p/_X9q4MWr4s)

在上面的程序中，`num`在`if`语句中初始化。需要注意的一点是，`num`仅可从`if`和`else`内部访问。即`num`的范围仅限于`if else`代码块，如果我们尝试从`if`或`else`外部访问`num`，编译器会报错。

让我们再写一个使用`else if`的程序。

```golang
package main

import (  
    "fmt"
)

func main() {  
    num := 99
    if num <= 50 {
        fmt.Println("number is less than or equal to 50")
    } else if num >= 51 && num <= 100 {
        fmt.Println("number is between 51 and 100")
    } else {
        fmt.Println("number is greater than 100")
    }

}
```
在上面的程序中，如果`else if num >= 51 && num <= 100`为真，那么程序将输出`number is between 51 and 100`

## 注意事项
`else`语句应该在`if`语句结束的`}`之后的同一行开始。如果不是，编译器会报错。

让我们通过一个程序来理解这一点。

```golang
package main

import (  
    "fmt"
)

func main() {  
    num := 10
    if num % 2 == 0 { //checks if number is even
        fmt.Println("the number is even") 
    }  
    else {
        fmt.Println("the number is odd")
    }
}
```
[Run in playground](https://play.golang.org/p/RYNqZZO2F9)

在上面的程序中，`else`语句没有在`if`语句接近`}`之后的同一行开始。相反，它从下一行开始。 Go 中不允许这样做，如果运行此程序，编译器将输出错误，
```plain
main.go:12:5: syntax error: unexpected else, expecting }  
```

原因是 Go 是自动插入分号的。你可以从这个链接查看有关分号插入规则的信息https://golang.org/ref/spec#Semicolons。

在规则中，如果`}`是该行最后的一个标记，go 将会在之后插入分号。因此，在`if`语句的`}`之后会自动插入分号。

所以我们的程序实际是下面这样的，

```golang
if num%2 == 0 {  
      fmt.Println("the number is even") 
};  //semicolon inserted by Go
else {  
      fmt.Println("the number is odd")
}
```

因为`{...} else {...}`是一个语句，所以在它的中间不应该有分号。因此，需要将`else`放在`}后的同一行中。
                                                                              
我已经通过在`if`语句的`}`之后插入`else`来重写程序，以防止自动分号插入。

```golang
package main

import (  
    "fmt"
)

func main() {  
    if num := 10; num % 2 == 0 { //checks if number is even
        fmt.Println("the number is even") 
    } else {
        fmt.Println("the number is odd")
    }
}
```
[Run in playground](https://play.golang.org/p/hv_27vbIBC)

现在编译器可以正常执行了。







