> * 原文地址：[Part 14: Strings](https://golangbot.com/strings/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

Go 中的字符串值得特别关注，因为与其他语言相比，它们的实现方式不太一样。

## 什么是字符串
Go 中的字符串是一个字节切片。可以通过将内容放在在""内来创建字符串。来看一个简单的例子，我们创建一个字符串并打印它。

```golang
package main

import (  
    "fmt"
)

func main() {  
    name := "Hello World"
    fmt.Println(name)
}
```
[Run in playground](https://play.golang.org/p/o9OVDgEMU0)

上面的程序将会输出`Hello World`。

Go 中的字符串兼容 Unicode，并且是 UTF-8 编码的。

## 访问字符串的单个字节
由于字符串是一个字节切片，因此可以访问字符串的每个字节。

```golang
package main

import (  
    "fmt"
)

func printBytes(s string) {  
    for i:= 0; i < len(s); i++ {
        fmt.Printf("%x ", s[i])
    }
}

func main() {  
    name := "Hello World"
    printBytes(name)
}
```
[Run in playground](https://play.golang.org/p/XbJO2b0ZDW)

在上面程序的第 8 行中，`len(s)`返回字符串中的字节数，我们使用 for 循环以十六进制表示法打印这些字节。 ％x 是十六进制的格式说明符。上述程序输出`48 65 6c 6c 6f 20 57 6f 72 6c 64`。这些是"Hello World"的[Unicode UT8](https://mothereff.in/utf-8#Hello%20World)码点值。对 Unicode 和 UTF-8 有基本的了解才能更好地理解字符串。我建议阅读[https://naveenr.net/unicode-character-set-and-utf-8-utf-16-utf-32-encoding/](https://naveenr.net/unicode-character-set-and-utf-8-utf-16-utf-32-encoding/)以了解什么是 Unicode 和 UTF-8。

让我们稍微修改上面的程序来打印字符串的字符。

```golang
package main

import (  
    "fmt"
)

func printBytes(s string) {  
    for i:= 0; i < len(s); i++ {
        fmt.Printf("%x ", s[i])
    }
}


func printChars(s string) {  
    for i:= 0; i < len(s); i++ {
        fmt.Printf("%c ",s[i])
    }
}

func main() {  
    name := "Hello World"
    printBytes(name)
    fmt.Printf("\n")
    printChars(name)
}
```
[Run in playground](https://play.golang.org/p/Jss0HG1q80)

在第 16 行的`printChars`方法中，％c 格式符用于打印字符串的字符。该程序输出

```plain
48 65 6c 6c 6f 20 57 6f 72 6c 64  
H e l l o   W o r l d  
```

虽然上面的程序看起来像是访问一个字符串的各个字符的合法方式，但这有一个严重的 bug。我们给这段代码打印一些断点信息，看看哪里做错了。

```golang
package main

import (  
    "fmt"
)

func printBytes(s string) {  
    for i:= 0; i < len(s); i++ {
        fmt.Printf("%x ", s[i])
    }
}

func printChars(s string) {  
    for i:= 0; i < len(s); i++ {
        fmt.Printf("%c ",s[i])
    }
}

func main() {  
    name := "Hello World"
    printBytes(name)
    fmt.Printf("\n")
    printChars(name)
    fmt.Printf("\n")
    name = "Señor"
    printBytes(name)
    fmt.Printf("\n")
    printChars(name)
}
```
[Run in playground](https://play.golang.org/p/UQOVvRVaFH)

上面的程序输出，

```plain
48 65 6c 6c 6f 20 57 6f 72 6c 64  
H e l l o   W o r l d  
53 65 c3 b1 6f 72  
S e Ã ± o r  
```

在上面程序的第 28 行中，我们试图打印 Señor 的字符但是输出了错误的 S e Ã ± o r。为什么这个程序会破坏`Señor`，其实它跟 Hello World 差不多啊。原因是ñ的 Unicode 的码点是 U + 00F1，其[UTF-8 编码]([https://mothereff.in/utf-8#%C3%B1](https://mothereff.in/utf-8#%C3%B1)
)占用 2 个字节 c3 和 b1。我们尝试打印字符，它以每个字符的码点都是 1 个字节长，所以结果是错误的。在 UTF-8 编码中，字符的码点可以超过 1 个字节。那么我们如何解决这个问题呢。这就是`rune`发挥作用的地方。

## rune
rune 是 Go 中的内置类型，它是 int32 的别名。rune 表示一个 Unicode 码点。码点占用的字节数无关紧要，它可以用 rune 表示。让我们修改上面的程序，使用 rune 打印字符。

```golang
package main

import (  
    "fmt"
)

func printBytes(s string) {  
    for i:= 0; i < len(s); i++ {
        fmt.Printf("%x ", s[i])
    }
}

func printChars(s string) {  
    runes := []rune(s)
    for i:= 0; i < len(runes); i++ {
        fmt.Printf("%c ",runes[i])
    }
}

func main() {  
    name := "Hello World"
    printBytes(name)
    fmt.Printf("\n")
    printChars(name)
    fmt.Printf("\n\n")
    name = "Señor"
    printBytes(name)
    fmt.Printf("\n")
    printChars(name)
}
```
[Run in playground](https://play.golang.org/p/t4z-f8I-ih)

在上述程序的第 14 行中，字符串被转换为 rune 切片。然后我们遍历它并显示字符。该程序输出，

```plain
48 65 6c 6c 6f 20 57 6f 72 6c 64  
H e l l o   W o r l d 

53 65 c3 b1 6f 72  
S e ñ o r  
```

上面的输出正是我们想要的。

## for range 循环一个字符串
上述程序是迭代字符串各个 rune 的一个不错的方式。但是 Go 可以使用 for range 循环为我们提供了一种更简单的方法。

```golang
package main

import (  
    "fmt"
)

func printCharsAndBytes(s string) {  
    for index, rune := range s {
        fmt.Printf("%c starts at byte %d\n", rune, index)
    }
}

func main() {  
    name := "Señor"
    printCharsAndBytes(name)
}
```
[Run in playground](https://play.golang.org/p/BPpQ0dZr8W)

在上述程序的第 8 行中，使用`for range`循环迭代字符串。循环返回 rune 开始的字节的位置以及 rune。该程序将输出

```plain
S starts at byte 0  
e starts at byte 1  
ñ starts at byte 2
o starts at byte 4  
r starts at byte 5  
```

从上面的输出可以清楚地看到`ñ`占用了 2 个字节:)。

## 用字节切片构造字符串
```golang
package main

import (  
    "fmt"
)

func main() {  
    byteSlice := []byte{0x43, 0x61, 0x66, 0xC3, 0xA9}
    str := string(byteSlice)
    fmt.Println(str)
}
```
[Run in playground](https://play.golang.org/p/Vr9pf8X8xO)

上面程序中的`byteSlice`包含字符串"Café"的 [UTF-8 编码](https://mothereff.in/utf-8#Caf%C3%A9)的十六进制字节。该程序输出了`Café`。

如果我们有十六进制值等值的十进制，该怎么让上述程序运行？我们来看看。

```golang
package main

import (  
    "fmt"
)

func main() {  
    byteSlice := []byte{67, 97, 102, 195, 169}//decimal equivalent of {'\x43', '\x61', '\x66', '\xC3', '\xA9'}
    str := string(byteSlice)
    fmt.Println(str)
}
```
[Run in playground](https://play.golang.org/p/jgsRowW6XN)
上面的程序也输出`Café`。

## 用 rune 切片构造一个字符串
```golang
package main

import (  
    "fmt"
)

func main() {  
    runeSlice := []rune{0x0053, 0x0065, 0x00f1, 0x006f, 0x0072}
    str := string(runeSlice)
    fmt.Println(str)
}
```
[Run in playground](https://play.golang.org/p/m8wTMOpYJP)

在上面的程序中，`runeSlice`包含字符串 Señor 的十六进制的 Unicode 编码。该程序输出`Señor`。

## 字符串的长度
[utf8 package](https://golang.org/pkg/unicode/utf8/#RuneCountInString) 的`func RuneCountInString(s string) (n int)`函数用于查找字符串的长度。此方法将字符串作为参数，并返回其中的 rune 数。

```golang
package main

import (  
    "fmt"
    "unicode/utf8"
)



func length(s string) {  
    fmt.Printf("length of %s is %d\n", s, utf8.RuneCountInString(s))
}
func main() {  

    word1 := "Señor" 
    length(word1)
    word2 := "Pets"
    length(word2)
}
```
[Run in playground](https://play.golang.org/p/QGYlHmF7tn)

程序输出，

```plain
length of Señor is 5  
length of Pets is 4  
```

## 字符串是不可变的
字符串在 Go 中是不可被改变的。一旦创建好字符串后，就无法修改它。

```golang
package main

import (  
    "fmt"
)

func mutate(s string)string {  
    s[0] = 'a'//any valid unicode character within single quote is a rune 
    return s
}
func main() {  
    h := "hello"
    fmt.Println(mutate(h))
}
```
[Run in playground](https://play.golang.org/p/bv4SlSd_hp)

在上述代码的第 8 行，我们尝试将字符串的第一个字符修改为'a'。这是不允许的，因为字符串是不可变的，因此程序抛出错误`main.go:8: cannot assign to s[0]`

要解决字符串不可修改的问题，需要将字符串转换为 rune 切片。然后，该切片随修改而修改，并转换返回新的字符串。

```golang
package main

import (  
    "fmt"
)

func mutate(s []rune) string {  
    s[0] = 'a' 
    return string(s)
}

func main() {  
    h := "hello"
    fmt.Println(mutate([]rune(h)))
}
```
[Run in playground](https://play.golang.org/p/GL1cm17IP1)

在上述程序的第 7 行中，`mutate`函数接受 rune 切片作为参数。然后它将切片的第一个元素更改为'a'，再将 rune 转换回字符串并返回它。程序的第 13 行调用了此方法，`h`被转换为 rune 切片并传递给`mutate`。该程序输出将`aello`

我在 github 中创建了一个程序，其中包含我们这节讨论的所有内容。您可以在这里[下载](https://github.com/golangbot/stringsexplained)。













