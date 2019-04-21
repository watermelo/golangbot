> * 原文地址：[Part 9: Loops](https://golangbot.com/loops/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

循环语句被用来重复执行一块代码。

`for`是 Go 中唯一的循环语句。 Go 没有`while`或`do while`循环，这些循环存在于其他如 C 语言中。

## for 循环的语法

```golang
for initialisation; condition; post {  
}
```

初始化语句只执行一次。初始化循环后，将检查条件。如果条件的计算结果为`true`，则执行`{}`内的循环体，然后执行`post`语句。`post`语句将在每次成功迭代循环后执行。执行`post`语句后，将重新检查条件。如果是`true`，则循环将继续执行，否则`for`循环终止。

在 Go 中，这三个组件叫初始化，`condition`和`post`都是可选的。让我们看一个例子来更好地理解 for 循环。

## 例子
我们编写一个程序，使用 for 循环打印从 1 到 10 的所有数字。

```golang
package main

import (  
    "fmt"
)

func main() {  
    for i := 1; i <= 10; i++ {
        fmt.Printf(" %d",i)
    }
}
```

[Run in playground](https://play.golang.org/p/mV6Zgcx2DY)

在上面的程序中，`i`被初始化为 1。条件语句将检查`i <= 10`。如果条件为满足，则打印`i`的值，否则循环终止，`post`语句在每次迭代结束时将`i`递增 1。一旦大于 10，循环终止。

程序将输出，`1 2 3 4 5 6 7 8 9 10`

在 for 循环中声明的变量仅在循环范围内可用。因此我不能在代码块外获取`i`。

## break
break 语句用于在正常执行之前终止 for 循环，并执行 for 循环之后的代码行。

让我们编写一个程序，使用 break 并打印 1 到 5 的数字。

```golang
package main

import (  
    "fmt"
)

func main() {  
    for i := 1; i <= 10; i++ {
        if i > 5 {
            break //loop is terminated if i > 5
        }
        fmt.Printf("%d ", i)
    }
    fmt.Printf("\nline after for loop")
}
```
[Run in playground](https://play.golang.org/p/sujKy92f--)

在上述程序中，每次迭代都检查`i`的值。如果`i`大于 5，则执行 break 并终止循环。然后执行 for 循环之后的 print 语句。以上程序将输出，
```plain
1 2 3 4 5  
line after for loop 
```

## cotinue
continue 语句用于跳过当前 for 循环的迭代。在 continue 语句之后的代码都不会在当前迭代中执行。循环将继续下一次迭代。

让我们编写一个程序，使用 continue 来打印从 1 到 10 的所有奇数。

```golang
package main

import (  
    "fmt"
)

func main() {  
    for i := 1; i <= 10; i++ {
        if i%2 == 0 {
            continue
        }
        fmt.Printf("%d ", i)
    }
}
```
[Run in playground](https://play.golang.org/p/DRLN2ZHwVS)

在上面的程序中，`if i%2 == 0`检查`i`除以 2 是否为 0。如果为 0 则为偶数并执行`continue`语句，执行下一次迭代环。因此，不会调用`continue`之后的`print`语句，循环进入下一次迭代。上述程序的输出为`1 3 5 7 9`

## 嵌套 for 循环
在循环中包含另一个 for 循环的 for 循环称为嵌套 for 循环。

让我们通过编写一个打印下面序列的程序来理解嵌套 for 循环。

```plain
*
**
***
****
*****
```

下面的程序使用嵌套 for 循环来打印序列。变量 n 存储序列的行数，在我们的例子中`n`是 5。外部 for 循环从 0 到 4 迭代`i`，内部 for 循环将 j 从 0 迭代到`i`的当前值。内部循环迭代打印*，外部循环在每次迭代结束时换行。运行此程序，将看到上述打印的序列。

```golang
package main

import (  
    "fmt"
)

func main() {  
    n := 5
    for i := 0; i < n; i++ {
        for j := 0; j <= i; j++ {
            fmt.Print("*")
        }
        fmt.Println()
    }
}
```

[Run in playground](https://play.golang.org/p/0rq8fWjVDLb)

## 标签 label
标签可用于从内部 for 循环内部打破外部循环。让我们通过一个简单的例子来理解我的意思。（译者注：Go 语言中可以用到 label 的有 goto，break，continue）

```golang
package main

import (  
    "fmt"
)

func main() {  
    for i := 0; i < 3; i++ {
        for j := 1; j < 4; j++ {
            fmt.Printf("i = %d , j = %d\n", i, j)
        }

    }
}
```
[Run in playground](https://play.golang.org/p/BnCKho2x5hM)

理解上面的代码不难，上述程序将打印，
```plain
i = 0 , j = 1  
i = 0 , j = 2  
i = 0 , j = 3  
i = 1 , j = 1  
i = 1 , j = 2  
i = 1 , j = 3  
i = 2 , j = 1  
i = 2 , j = 2  
i = 2 , j = 3  
```

如果我们想在`i`和`j`相等时停止打印怎么办？要做到这一点，我们需要在外部 for 循环中断。当`i`和`j`相等时，在内部 for 循环中添加一个 break 只会从内部 for 循环中中断。

```golang
package main

import (  
    "fmt"
)

func main() {  
    for i := 0; i < 3; i++ {
        for j := 1; j < 4; j++ {
            fmt.Printf("i = %d , j = %d\n", i, j)
            if i == j {
                break
            }
        }

    }
}
```
[Run in playground](https://play.golang.org/p/uMjbF8Ii41d)

在上面的程序中，当`i`和`j`相等时，我在内部 for 循环中添加了一个 break。这将仅从内部 for 循环中断，外循环将继续，该程序将打印，

```plain
i = 0 , j = 1  
i = 0 , j = 2  
i = 0 , j = 3  
i = 1 , j = 1  
i = 2 , j = 1  
i = 2 , j = 2  
```

这不是期望的输出。当`i`和`j`相等时，即当它们都等于 1 时，我们需要停止打印。

这是标签发挥作用的地方。标签可用于从外循环中断开。让我们用标签重写上面的程序。

```golang
package main

import (  
    "fmt"
)

func main() {  
outer:  
    for i := 0; i < 3; i++ {
        for j := 1; j < 4; j++ {
            fmt.Printf("i = %d , j = %d\n", i, j)
            if i == j {
                break outer
            }
        }

    }
}
```

[Run in plaground](https://play.golang.org/p/BI10Rmp_Z3y)

在上面的程序中，我们在第 8 行为外部 for 循环添加了一个标签。我们通过中断指定标签来结束外部 for 循环。当`i`和`j`相等时，该程序将停止打印。该程序将输出

```plain
i = 0 , j = 1  
i = 0 , j = 2  
i = 0 , j = 3  
i = 1 , j = 1  
```

## 更多例子
让我们编写一些代码来熟悉 for 循环的各种用法。

下面的程序打印从 0 到 10 的所有偶数。

```golang
package main

import (  
    "fmt"
)

func main() {  
    i := 0
    for ;i <= 10; { // initialisation and post are omitted
        fmt.Printf("%d ", i)
        i += 2
    }
}
```
[Run in playground](https://play.golang.org/p/PNXliGINku)

我们已经知道 for 循环有三个组件，即`initialisation`，`condition`和`post`是可选的。在上面的程序中，省略了`initialisation`和`post`。我在 for 循环之外初始化`i`为 0。只要`i <= 10`，就会执行循环。`i`在 for 循环中增加 2。上述程序输出`0 2 4 6 8 10`。

也可以省略上述程序的 for 循环中的分号。这种格式可以被认为是 while 循环的替代方案。上述程序可以改写为，

```golang
package main

import (  
    "fmt"
)

func main() {  
    i := 0
    for i <= 10 { //semicolons are ommitted and only condition is present
        fmt.Printf("%d ", i)
        i += 2
    }
}
```
[Run in playground](https://play.golang.org/p/UYiz-Wtnoj)

我们也可以在 for 循环中声明和操作多个变量。来编写一个程序，在 for 语句中使用多个变量声明并打印下面的序列。

```plain
10 * 1 = 10  
11 * 2 = 22  
12 * 3 = 36  
13 * 4 = 52  
14 * 5 = 70  
15 * 6 = 90  
16 * 7 = 112  
17 * 8 = 136  
18 * 9 = 162  
19 * 10 = 190  
```

```golang
package main

import (  
    "fmt"
)

func main() {  
    for no, i := 10, 1; i <= 10 && no <= 19; i, no = i+1, no+1 { //multiple initialisation and increment
        fmt.Printf("%d * %d = %d\n", no, i, no*i)
    }

}
```
[Run in playground](https://play.golang.org/p/e7Pf0UDjj0)

在上面的程序中，`no`和`i`被声明并初始化为 10 和 1。它们在每次迭代结束时递增 1。在`condition`中使用布尔运算符`&&`以确保`i`小于或等于 10 并且`no`小于或等于 19。

## 无限循环
无限循环的语法为， 

```golang
for {  
}
```

以下程序将一直打印 Hello World 且不会结束。

```golang
package main

import "fmt"

func main() {  
    for {
        fmt.Println("Hello World")
    }
}
```

如果你在 playground 上运行该程序，将会报错`process took too long`。请尝试在本地系统中运行它以无限地打印`Hello World`。

还有`range`可以用 for 循环以进行数组的迭代操作。我们了解数组时，再介绍这一点。



