> * 原文地址：[Part 31: Custom Errors](https://golangbot.com/custom-errors/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

在上一个教程中，我们学习了在 Go 语言中的```error```是如何表示的以及怎么用标准库处理```error```。我们还学习了如何从标准库中提取更多的```error```信息。

本教程介绍如何在我们的函数和包中使用自己定义的```error```，我们将使用标准库使用的相同技术来提供有关我们的自定义```error```的更多信息。

## 使用```New```函数创建自定义```error```

创建自定义```error```最简单的方法是使用```errors```包的[New](https://golang.org/pkg/errors/#New)函数。

在我们使用```New```函数创建自定义```error```之前，我们先来了解它是如何实现的。下面提供了[errors 包](https://golang.org/src/errors/errors.go?s=293:320#L1)中```New```函数的实现。
```golang
// Package errors implements functions to manipulate errors.
  package errors

  // New returns an error that formats as the given text.
  func New(text string) error {
      return &errorString{text}
  }

  // errorString is a trivial implementation of error.
  type errorString struct {
      s string
  }

  func (e *errorString) Error() string {
      return e.s
  }
```
实现非常简单。```errorString```是一个带有单个字符串```s```的结构类型。```error```接口的```Error```方法是在第 14 行使用```errorString```指针接收器实现的。

第 5 行的```New```函数接受一个字符串参数，使用该参数创建一个```errorString```类型的值并返回它的地址，一个新的```error```就完成了。

现在我们知道了```New```函数的工作原理，让我们在自己的程序中使用它来创建自定义```error```。

我们将创建一个计算圆的面积的简单程序，如果半径为负，则返回```error```。

```golang
package main

import (
    "errors"
    "fmt"
    "math"
)

func circleArea(radius float64) (float64, error) {
    if radius < 0 {
        return 0, errors.New("Area calculation failed, radius is less than zero")
    }
    return math.Pi * radius * radius, nil
}

func main() {
    radius := -20.0
    area, err := circleArea(radius)
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Printf("Area of circle %0.2f", area)
}
```
[Run in playground](http://play.flysnow.org/p/7jCd9RxzQLl)

在上面的程序中，我们在第 10 行检查半径是否小于零。如果是，则返回 0 以及相应的```error```内容。在第 13 行中，如果半径大于 0，则计算面积并返回值为```nil```的```error```。

在 main 函数中，我们在第 19 行检查```error```是否为```nil```。如果不是```nil```，就打印错误并返回，否则打印区域的面积。

在这个程序中，半径小于零，因此它将打印，
```plain
Area calculation failed, radius is less than zero
```

## 使用```Errorf```为```error```增加更多的信息

上面的程序效果还不错，但是如果我们要想打印确切的错误信息就有点吃力了。这种场景```fmt```包的```Errorf```函数就派上用场了。此函数使用字符串格式化参数，并返回一个字符串作为```error```的值。

让我们来试试```Errorf```函数，

```golang
package main

import (
    "fmt"
    "math"
)

func circleArea(radius float64) (float64, error) {
    if radius < 0 {
        return 0, fmt.Errorf("Area calculation failed, radius %0.2f is less than zero", radius)
    }
    return math.Pi * radius * radius, nil
}

func main() {
    radius := -20.0
    area, err := circleArea(radius)
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Printf("Area of circle %0.2f", area)
}
```
[Run in playground](http://play.flysnow.org/p/0Nciss3ojbw)

在上面的程序中，第 10 行使用了```Errorf```， 打印导致错误的实际半径值，运行此程序将输出，
```plain
Area calculation failed, radius -20.00 is less than zero
```
## 使用结构类型和字段提供有关```error```的更多信息

也可以使用实现错误接口的结构类型作为```error```。这为我们提供了更多的灵活性。在我们的示例中，如果我们想要查看错误，唯一的方法是解析```Area calculation failed, radius -20.00 is less than zero```。这不是一种合适的方法，因为如果描述发生变化，我们的代码逻辑就得修改。

我们将使用前一个教程中所提到的“断言结构类型并从结构类型的字段中获取更多信息”方式，使用结构字段来提供```err```和```radius```。我们将创建一个实现```error```接口的结构类型，并使用其字段提供有关的更多信息。

第一步是创建一个结构类型来表示```error```。类型的命名约定是```Error```结尾。所以我们将结构类型命名为```areaError```
```golang
type areaError struct {
    err    string
    radius float64
}
```
上面的结构类型有一个字段```radius```，它存储了负责该```error```的半径值，而```err```字段则存储了实际的错误内容。

下一步是实现```error```接口。
```golang
func (e *areaError) Error() string {
    return fmt.Sprintf("radius %0.2f: %s", e.radius, e.err)
}
```
在上面的代码片段中，我们使用指针接收者```* areaError```实现错误接口的```Error```方法。该方法打印```radius```和```err```描述。

让我们通过编写```main```函数和```circleArea```函数来完成程序。
```golang
package main

import (
    "fmt"
    "math"
)

type areaError struct {
    err    string
    radius float64
}

func (e *areaError) Error() string {
    return fmt.Sprintf("radius %0.2f: %s", e.radius, e.err)
}

func circleArea(radius float64) (float64, error) {
    if radius < 0 {
        return 0, &areaError{"radius is negative", radius}
    }
    return math.Pi * radius * radius, nil
}

func main() {
    radius := -20.0
    area, err := circleArea(radius)
    if err != nil {
        if err, ok := err.(*areaError); ok {
            fmt.Printf("Radius %0.2f is less than zero", err.radius)
            return
        }
        fmt.Println(err)
        return
    }
    fmt.Printf("Area of rectangle1 %0.2f", area)
}
```
[Run in playgroud](http://play.flysnow.org/p/G38vOhidIoy)

在上面的程序中，第 17 行的```circleArea```用于计算圆的面积。此函数首先检查半径是否小于零，如果是，则使用该错误的半径和相应的错误内容描述去创建```areaError```，然后在第 19 行返回```areaError```的地址和错误内容。因此，我们提供了更多的```error```信息，在这个例子中，使用```error```结构的字段实现了自定义。

如果半径不是负数，则此函数计算并返回面积和```nil```

在第 26 行的```main```函数中，我们试图计算半径为-20 的圆面积。由于半径小于零，因此将返回错误。

我们在第 27 行检查```err```是否为```nil```，并在下一行中断言```err```是```* areaError```类型。如果是```* areaError类型```，我们在 29 行使用```err.radius```获取造成```error```的```radius```，然后打印自定义错误内容并从程序返回。

输出，
```plain
Radius -20.00 is less than zero
```

现在让我们使用上一个教程中描述的第二个策略，并使用自定义错误类型的方法来提供有关```error```的更多信息。

## 使用结构类型的方法提供有关```error```的更多信息

在本节中，我们将编写一个计算矩形区域的程序。如果长度或宽度小于 0，该程序将打印错误。

第一步是创建一个表示```error```的结构。
```golang
type areaError struct {
    err    string //error description
    length float64 //length which caused the error
    width  float64 //width which caused the error
}
```
上面的错误结构类型包含错误描述字段```err```以及长度```length```和宽度```width```。

现在我们已经定义好了```error```类型，让我们实现```error```接口并在类型上添加几个方法以提供有关的更多信息。

```golang
func (e *areaError) Error() string {
    return e.err
}

func (e *areaError) lengthNegative() bool {
    return e.length < 0
}

func (e *areaError) widthNegative() bool {
    return e.width < 0
}
```

在上面的代码片段中，我们从```Error```方法返回错误的描述```e.err```。当```length```小于 0 时，```lengthNegative```方法返回```true```，而当```width```小于 0 时，```widthNegative```方法返回```true```。这两种方法提供了有关信息，在这种情况下，它们表示区域计算是否因为长度为负或宽度为负而失败。因此，我们使用```error```结构类型的方法来提供有关的更多信息。

下一步是实现面积计算功能。

```golang
func rectArea(length, width float64) (float64, error) {
    err := ""
    if length < 0 {
        err += "length is less than zero"
    }
    if width < 0 {
        if err == "" {
            err = "width is less than zero"
        } else {
            err += ", width is less than zero"
        }
    }
    if err != "" {
        return 0, &areaError{err, length, width}
    }
    return length * width, nil
}
```
上面的```rectArea```函数检查长度或宽度是否小于 0，如果是，则返回 0 和```error```，否则返回矩形面积和```nil```。
让我们通过创建```main```函数来完成整个程序。
```golang
func main() {
    length, width := -5.0, -9.0
    area, err := rectArea(length, width)
    if err != nil {
        if err, ok := err.(*areaError); ok {
            if err.lengthNegative() {
                fmt.Printf("error: length %0.2f is less than zero\n", err.length)

            }
            if err.widthNegative() {
                fmt.Printf("error: width %0.2f is less than zero\n", err.width)

            }
            return
        }
        fmt.Println(err)
        return
    }
    fmt.Println("area of rect", area)
}
```
在```main```函数中，我们检查```err```是否为```nil```。 如果它不是```nil```，我们在下一行断言```* areaError```。然后使用```lengthNegative```和```widthNegative```方法，检查错误是否是因为长度为负或宽度为负。我们打印相应的```error```内容并从程序返回。因此，我们使用结构类型的方法来提供有关的更多信息。

如果没有错误，将打印矩形面积。

最后贴出完整的程序供参考。
```golang
package main

import "fmt"

type areaError struct {
    err    string  //error description
    length float64 //length which caused the error
    width  float64 //width which caused the error
}

func (e *areaError) Error() string {
    return e.err
}

func (e *areaError) lengthNegative() bool {
    return e.length < 0
}

func (e *areaError) widthNegative() bool {
    return e.width < 0
}

func rectArea(length, width float64) (float64, error) {
    err := ""
    if length < 0 {
        err += "length is less than zero"
    }
    if width < 0 {
        if err == "" {
            err = "width is less than zero"
        } else {
            err += ", width is less than zero"
        }
    }
    if err != "" {
        return 0, &areaError{err, length, width}
    }
    return length * width, nil
}

func main() {
    length, width := -5.0, -9.0
    area, err := rectArea(length, width)
    if err != nil {
        if err, ok := err.(*areaError); ok {
            if err.lengthNegative() {
                fmt.Printf("error: length %0.2f is less than zero\n", err.length)

            }
            if err.widthNegative() {
                fmt.Printf("error: width %0.2f is less than zero\n", err.width)

            }
            return
        }
    }
    fmt.Println("area of rect", area)
}
```
[Run in playground](http://play.flysnow.org/p/VRSrXJLzfbq)
程序输出，
```plain
error: length -5.00 is less than zero
error: width -9.00 is less than zero
```
我们已经看到了```error```处理教程中描述的三种方法中的两种的示例，以提供有关的更多信息。

第三种方式使用直接比较的方法比较简单。我会留下它作为练习，让您弄清楚如何使用此策略来提供有关我们的自定义```error```的更多信息。
