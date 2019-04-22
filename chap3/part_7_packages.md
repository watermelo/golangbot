> * 原文地址：[Part 7: Packages](https://golangbot.com/packages/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

## 什么是包以及为什么要使用它
到目前为止，我们看到的 go 代码只有一个文件，其中有一个 main 函数和其他几个函数。在实际场景中，将所有源代码写入单个文件不是一个好方法。复用和维护这种代码将变得非常艰难，包就是用来解决这些问题的。

包可以使代码更好的复用和可读，也可以使代码解耦，因此使得应用程序很容易维护。

例如，假设我们正在创建一个 go 图像处理的应用程序，它提供了图像裁剪，锐化，模糊和色彩增强等功能。组织此应用程序的一种方法是将与功能相关的所有代码分组到自己的包中。例如，裁剪可以是单个包，锐化可以是另一个包。这样做的优点是，色彩增强功能可能需要一些锐化功能。色彩增强代码可以简单地导入（我们将马上讨论包导入）锐化包并使用其功能。这样代码就变得易于复用。

我们将逐步创建一个计算矩形区域和对角线的应用程序，借此应用程序让我们更好地了解包的作用。

## main 函数和 main 包
每个可执行的 go 应用程序都必须包含 main 函数。此函数是执行的入口。 main 函数应该放在 main 包中。

每个 go 源代码文件的第一行是`package packagename`，该语法用来指定该文件属于哪个包。

让我们开始为我们的应用程序创建 main 函数和 main 包。在 go 工作区的 src 文件夹中创建一个文件夹，并将其命名为 geometry。在 geometry 文件夹中创建文件 geometry.go。

geometry.go 的代码如下，

```golang
//geometry.go
package main 

import "fmt"

func main() {  
    fmt.Println("Geometrical shape properties")
}
```

代码`package main`指定此文件属于 main 包。`import "packagename"`语句用于导入现有的包。在该例子中，我们导入了包含`Println`方法的 fmt 包。然后有一个 main 函数，打印了`Geometrical shape properties`，

通过输入`go install geometry`来编译上面的程序。此命令在 geometry 文件夹中搜索具有 main 函数的文件。在该例子中，它会找到 geometry.go。然后编译它并在工作区的`bin`文件夹内生成一个名为 geometry 的二进制文件（在 windows 的情况下为 geometry.exe）。现在工作区结构将是

```plain
src  
    geometry
            gemometry.go
bin  
    geometry
```

我们输入`workspacepath/bin/geometry`来运行程序。将 workspacepath 替换为 go 工作区的路径。此命令执行 bin 文件夹中的 geometry 二进制文件，你可以看到输出为`Geometrical shape properties`。

## 创建自定义包
我们以这样的方式来组织代码，让与矩形相关的所有功能都在 rectangle 包中。

我们来创建一个自定义包`rectangle`，它拥有计算矩形区域和对角线长度的两个函数。

包的源文件应放在自己的单独文件夹中，Go 中的一个约定是包的名称和此文件夹的名称相同。

因此，我们在 geometry 文件夹中创建一个名为 rectangle 的文件夹。rectangle 文件夹中的所有文件都应以`package rectangle`开头，因为它们都属于`rectangle`包。

在 rectangle 文件夹中创建一个 rectprops.go 文件，并添加以下代码，

```golang
//rectprops.go
package rectangle

import "math"

func Area(len, wid float64) float64 {  
    area := len * wid
    return area
}

func Diagonal(len, wid float64) float64 {  
    diagonal := math.Sqrt((len * len) + (wid * wid))
    return diagonal
}
```

在上面的代码中，我们创建了`Area`和`Diagonal`函数。矩形的面积是长度和宽度的乘积，矩形的对角线长度是长度和宽度的平方和的平方根。`math`包中的`Sqrt`函数用于计算平方根。

注意，函数名称`Area`和`Diagonal`都是大写字母开头，这是必要的，我们待会解释。

## 导入自定义包
要使用自定义包，必须先导入它。`import path`是导入自定义包的语法。我们必须指定自定义包相对于工作空间内的 src 文件夹的路径。我们当前的文件夹结构是

```plain
src  
   geometry
           geometry.go
           rectangle
                    rectprops.go
```

`import "geometry/rectangle"` 是导入 rectangle 包的方式。

给 geometry.go 再添加些代码，

```golang
//geometry.go
package main 

import (  
    "fmt"
    "geometry/rectangle" //importing custom package
)

func main() {  
    var rectLen, rectWidth float64 = 6, 7
    fmt.Println("Geometrical shape properties")
        /*Area function of rectangle package used
        */
    fmt.Printf("area of rectangle %.2f\n", rectangle.Area(rectLen, rectWidth))
        /*Diagonal function of rectangle package used
        */
    fmt.Printf("diagonal of the rectangle %.2f ",rectangle.Diagonal(rectLen, rectWidth))
}
```

上面的代码导入了 rectangle 包，并使用它的`Area`和`Diagonal`函数来计算矩形面积和对角线长度。`Printf`中的`％.2f`格式化符号是将浮点数截取为两位小数。该程序的输出是

```plain
Geometrical shape properties  
area of rectangle 42.00  
diagonal of the rectangle 9.22  
```

## 可导出命名
将 rectangle 包中的函数`Area`和`Diagonal`首字母大写，这在 Go 中有特殊意义。任何以大写字母开头的变量或函数都是 go 中的可导出命名，用这种命名方式的函数和变量能够被其他包访问。在该例子中，我们通过这种方式在 main 包中了访问`Area`和`Diagonal`行数。

如果把 rectprops.go 的函数名由`Area(len, wid float64)`更改为`area(len, wid float64)`，并在 geometry.go 中由`rectangle.Area(rectLen, rectWidth)`更改为`rectangle.area(rectLen, rectWidth)`。如果程序运行，编译器将抛出错误`geometry.go:11: cannot refer to unexported name rectangle.area`。因此，如果要访问其他包的函数，则应使用可导出的命名方式。

## init 函数
每个包都可以包含 init 函数。init 函数不应该有任何返回类型，也不应该有任何参数，在我们的源代码中也无法显式调用 init 函数。 init 函数如下所示

```golang
func init() {  
}
```

init 函数用于执行初始化任务，也可用于在执行开始之前验证程序的正确性。

包的初始化顺序如下
1. 首先初始化包级别变量
2. 接下来调用 init 函数。一个包可以有多个 init 函数（在单个文件中或分布在多个文件中），并按照它们呈现给编译器的顺序调用它们。

如果包导入其他包，则首先初始化该导入的包。

即使从多个包导入同一个包，被导入的包也只会初始化一次。

让我们对我们的应用程序进行一些修改以理解 init 函数。

首先，我们将一个 init 函数添加到 rectprops.go 文件中。

```golang
//rectprops.go
package rectangle

import "math"  
import "fmt"

/*
 * init function added
 */
func init() {  
    fmt.Println("rectangle package initialized")
}
func Area(len, wid float64) float64 {  
    area := len * wid
    return area
}

func Diagonal(len, wid float64) float64 {  
    diagonal := math.Sqrt((len * len) + (wid * wid))
    return diagonal
}
```

我们添加了一个简单的 init 函数，它只打印`rectangle package initialised`

现在来修改一下 main 包，众所周知矩形的长度和宽度应该大于零。我们可以使用 geometry.go 文件中的 init 函数和包级别变量来定义此检查。

将 geometry.go 修改为如下，

```golang
//geometry.go
package main 

import (  
    "fmt"
    "geometry/rectangle" //importing custom package
    "log"
)
/*
 * 1. package variables
*/
var rectLen, rectWidth float64 = 6, 7 

/*
*2. init function to check if length and width are greater than zero
*/
func init() {  
    println("main package initialized")
    if rectLen < 0 {
        log.Fatal("length is less than zero")
    }
    if rectWidth < 0 {
        log.Fatal("width is less than zero")
    }
}

func main() {  
    fmt.Println("Geometrical shape properties")
    fmt.Printf("area of rectangle %.2f\n", rectangle.Area(rectLen, rectWidth))
    fmt.Printf("diagonal of the rectangle %.2f ",rectangle.Diagonal(rectLen, rectWidth))
}
```

以下是对 geometry.go 所做的更改
1. `rectLen`和`rectWidth`变量从 main 函数级别移动到包级别。
2. 添加了一个 init 函数。如果`rectLen`或`rectWidth`小于零则使用`log.Fatal`函数打印日志并终止。

main 包的初始化顺序是
1. 首先初始化导入的包。因此 rectangle 包首先被初始化
2. 接下来初始化包级别变量`rectLen`和`rectWidth`
3. 调用 init 函数
4. 最后调用 main 函数

如果运行程序，将输出
```plain
rectangle package initialized  
main package initialized  
Geometrical shape properties  
area of rectangle 42.00  
diagonal of the rectangle 9.22  
```

正如所料，首先调用 rectangle 包的 init 函数，然后初始化包级变量`rectLen`和`rectWidth`，接下来调用 main 包的 init 函数，它检查`rectLen`和`rectWidth`是否小于零，如果小于零则打印日志并终止。我们会在单独的教程中详细了解 if 语句。现在你可以假设如果`rectLen < 0`就是判断`rectLen`是否小于 0，如果是，则程序将被终止。我们为`rectWidth`也写了一样的条件。在该例子中，两个条件都为`false`，程序继续执行。最后调用 main 函数。

再稍微修改一下该程序，以了解 init 函数的用法。

将 geometry.go 的`var rectLen, rectWidth float64 = 6, 7`更改为`var rectLen, rectWidth float64 = -6, 7`，`rectLen`被初始化为负数。

修改后运行会输出，

```plain
rectangle package initialized  
main package initialized  
2017/04/04 00:28:20 length is less than zero  
```

正常情况下，先初始化 rectangle 包，然后是 main 包中的包级别变量`rectLen`和`rectWidth`。`rectLen`是负数。因此，当 init 函数运行时，程序在打印`length is less than zero`后终止。

## 使用`_`符号
Go 中导入包并且不在代码中的任何位置使用它是非法的。如果你这样做，编译器会报错。这样做的原因是为了避免大量未使用的包显著增加编译时间。用以下代码替换 geometry.go 中的代码，

```golang
//geometry.go
package main 

import (   

     "geometry/rectangle" //importing custom package

)
func main() {

}
```

上面的程序将抛出错误`geometry.go:6: imported and not used: "geometry/rectangle"`

但是，当应用程序处于开发状态时导入各种包很常见，可能接下来不一定要使用这个包。 `_`符号可以用于这些场景。


以下代码可以使上述程序中正常运行，

```golang
package main

import (  
    "geometry/rectangle" 
)

var _ = rectangle.Area //error silencer

func main() {

}
```

该行`var _ = rectangle.Area`会使程序不会报错。如果我们不使用这个包，我们应该关注这些代码，并在应用程序开发结束时删除它们。因此，在程序开发阶段，可以在 import 语句之后用该方式避免报错。（译者注：一般用`_`存储不使用的变量等等，毕竟声明了如果不使用编译器会报错的）

有时我们导入一个包只是为了使用初始化功能，除此之外我们不需要使用包中的任何函数或变量。例如，我们可能需要调用 rectangle 包的 init 函数，即使在代码中的任何位置都不使用该包。在这种情况下也可以使用`_`符号，如下所示。

```golang
package main 

import (   

     _ "geometry/rectangle" 

)
func main() {

}
```

运行上面的程序将输出`rectangle package initialized`。我们已经成功初始化了包，即使它没有在代码中的任何地方被使用。（译者注：包导入还有`.`操作可以省略前缀包名直接调用该包的函数或者变量。别名操作也比较好理解，就是把前缀包名重命名，语法是 `import alias package`）




























