> * 原文地址：[Part 11: Arrays and Slices](https://golangbot.com/arrays-and-slices/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

这一章我们将学习 Go 语言中的数组和切片。

## 数组

数组是属于同一类型的元素的集合。例如，整数 5, 8, 9, 79, 76 的集合可以构成数组。Go 中的数组不允许混合不同类型的值，例如包含字符串和整数。

##### 数组的声明
数组的表示为`[n] T`。n 表示数组中元素的数量，T 表示每个元素的类型。元素的数量 n 也是类型的一部分（我们将在稍后更详细地讨论它。）

有多种声明数组的方法，我们一个一个来看看，

```golang
package main

import (  
    "fmt"
)


func main() {  
    var a [3]int //int array with length 3
    fmt.Println(a)
}
```
[Run in playground](https://play.golang.org/p/Zvgh82u0ej)
`var a [3]int` 声明了一个长度为 3 的整型数组，该数组所有的元素会被自动初始化为数组类型的零值。在这个例子中数组的类型为整数，所以 a 数组所有的元素会被初始化为 0。运行代码会输出，`[0 0 0]`

数组的索引从 0 开始到数组的长度减 1，我们来给数组 a 赋一些值，

```golang
package main

import (  
    "fmt"
)


func main() {  
    var a [3]int //int array with length 3
    a[0] = 12 // array index starts at 0
    a[1] = 78
    a[2] = 50
    fmt.Println(a)
}
```
[Run in playground](https://play.golang.org/p/WF0Uj8sv39)

赋值后，程序将输出`[12 78 50]`

我们也可以使用简短声明来创建一个数组。

```golang
package main 

import (  
    "fmt"
)

func main() {  
    a := [3]int{12, 78, 50} // short hand declaration to create array
    fmt.Println(a)
}
```
[Run in playground](https://play.golang.org/p/NKOV04zgI6)

该程序和上一个输出的一样`[12 78 50]`

在简短声明的时候，不需要为数组中的所有元素分配值。

```golang
package main

import (  
    "fmt"
)

func main() {  
    a := [3]int{12} 
    fmt.Println(a)
}
```
[Run in playground](https://play.golang.org/p/AdPH0kXRly)

上述代码的第 8 行`a := [3]int{12}`声明了一个长度为 3 的数组，但是只提供了一个值 12，剩下的两个元素将会被零值自动填充，程序将输出，`[12 0 0]`

在声明数组的时候，甚至可以不指定长度，而用`...`替代，该方法编译器会帮你计算出长度。来看看下面的代码，

```golang
package main

import (  
    "fmt"
)

func main() {  
    a := [...]int{12, 78, 50} // ... makes the compiler determine the length
    fmt.Println(a)
}
```
[Run in playground](https://play.golang.org/p/_fVmr6KGDh)

数组的大小是类型的一部分。因此`[5] int`和`[25] int`是不同的类型。因此，数组的大小是无法调整的。不要担心这个问题，因为切片就是为了解决这个问题。

```golang
package main

func main() {  
    a := [3]int{5, 78, 8}
    var b [5]int
    b = a //not possible since [3]int and [5]int are distinct types
}
```

[Run in playground](https://play.golang.org/p/kBdot3pXSB)

在上述程序的第 6 行，我们尝试把`[3]int`类型的值赋给`[5]int`，这是不允许的，所以编译器将会报错`main.go:6: cannot use a (type [3]int) as type [5]int in assignment.`

##### 数组是值传递类型

Go 中的数组是值传递类型而不是引用传递类型。这意味着当它们赋值给新变量时，会将原始数组的副本赋给新变量。如果对新变量进行了更改，则它不会改变原来的数组。

```golang
package main

import "fmt"

func main() {  
    a := [...]string{"USA", "China", "India", "Germany", "France"}
    b := a // a copy of a is assigned to b
    b[0] = "Singapore"
    fmt.Println("a is ", a)
    fmt.Println("b is ", b) 
}
```

[Run in playground](https://play.golang.org/p/-ncGk1mqPd)

上述程序的第 7 行，把`a`赋值给了`b`，第 8 行将`b`数组的第一个元素修改为 Singapore，这不会影响`a`。所以会输出，
```plain
a is [USA China India Germany France]  
b is [Singapore China India Germany France]  
```

类似地，当数组作为参数传递给函数时，它们按值传递，原数组也不会变。

```golang
package main

import "fmt"

func changeLocal(num [5]int) {  
    num[0] = 55
    fmt.Println("inside function ", num)

}
func main() {  
    num := [...]int{5, 6, 7, 8, 8}
    fmt.Println("before passing to function ", num)
    changeLocal(num) //num is passed by value
    fmt.Println("after passing to function ", num)
}
```
[Run in playground](https://play.golang.org/p/e3U75Q8eUZ)

上述程序的第 13 行，我们将数组`num`当作参数传递给了函数`changeLocal`。函数调用后将不会改变原`num`数组的值，程序输出，

```plain
before passing to function  [5 6 7 8 8]  
inside function  [55 6 7 8 8]  
after passing to function  [5 6 7 8 8]  
```

##### 数组的长度
数组的长度通过把数组当参数传递给`len`函数来计算。

```golang
package main

import "fmt"

func main() {  
    a := [...]float64{67.7, 89.8, 21, 78}
    fmt.Println("length of a is",len(a))

}
```
[Run in playground](https://play.golang.org/p/UrIeNlS0RN)

上述代码将输出，`length of a is 4`

##### 使用 range 迭代数组
可以用`for`循环来迭代一个数组的所有元素。

```golang
package main

import "fmt"

func main() {  
    a := [...]float64{67.7, 89.8, 21, 78}
    for i := 0; i < len(a); i++ { //looping from 0 to the length of the array
        fmt.Printf("%d th element of a is %.2f\n", i, a[i])
    }
}
```
[Run in playground](https://play.golang.org/p/80ejSTACO6)
上述代码使用了`for`循环迭代了数组的所有元素，该程序将输出，
```plain
0 th element of a is 67.70  
1 th element of a is 89.80  
2 th element of a is 21.00  
3 th element of a is 78.00  
```

Go 提供了一种更好，更简洁的方法，通过使用`for`循环的`range`形式迭代数组。 `range`返回索引和该索引处的值。让我们使用`range`重写上面的代码，并计算所有元素的和。

```golang
package main

import "fmt"

func main() {  
    a := [...]float64{67.7, 89.8, 21, 78}
    sum := float64(0)
    for i, v := range a {//range returns both the index and value
        fmt.Printf("%d the element of a is %.2f\n", i, v)
        sum += v
    }
    fmt.Println("\nsum of all elements of a",sum)
}
```

[Run in playground](https://play.golang.org/p/Ji6FRon36m)

上述代码的第 8 行，`for i, v := range a`就是该循环的形式。它将返回索引和该索引处的值。我们打印值并计算数组`a`的所有元素的和。该程序的输出是，
```plain
0 the element of a is 67.70  
1 the element of a is 89.80  
2 the element of a is 21.00  
3 the element of a is 78.00

sum of all elements of a 256.5  
```

在该例子中如果你只想要值，你可以使用`_`占位符替代返回索引的位置。

```golang
for _, v := range a { //ignores index  
}
```
类似的，值也可以被忽略。

##### 多维数组
目前位置我们创建的都是一维数组，当然也可以创建多维数组。

```golang
package main

import (  
    "fmt"
)

func printarray(a [3][2]string) {  
    for _, v1 := range a {
        for _, v2 := range v1 {
            fmt.Printf("%s ", v2)
        }
        fmt.Printf("\n")
    }
}

func main() {  
    a := [3][2]string{
        {"lion", "tiger"},
        {"cat", "dog"},
        {"pigeon", "peacock"}, //this comma is necessary. The compiler will complain if you omit this comma
    }
    printarray(a)
    var b [3][2]string
    b[0][0] = "apple"
    b[0][1] = "samsung"
    b[1][0] = "microsoft"
    b[1][1] = "google"
    b[2][0] = "AT&T"
    b[2][1] = "T-Mobile"
    fmt.Printf("\n")
    printarray(b)
}
```

[Run in playground](https://play.golang.org/p/InchXI4yY8)

在上面的程序中的第 17 中，使用简短语法声明了二维字符串数组`a`。第 20 行末尾的逗号是必要的，这是因为词法分析器根据简单的规则自动插入分号。如果您有兴趣了解更多，可以阅读https://golang.org/doc/effective_go.html#semicolons。

另一个二维数组`b`在第 23 行被声明。通过每个索引逐个添加字符串是初始化二维数组的另一种方法。

第 7 行中的`printarray`函数，使用两个`for range`循环来打印二维数组的内容。上述程序的输出是
```plain
lion tiger  
cat dog  
pigeon peacock 

apple samsung  
microsoft google  
AT&T T-Mobile  
```

这就是数组。尽管数组似乎足够灵活，但它具有固定长度的限制，无法增加数组的长度，而切片可以自动扩充长度。事实上，在 Go 中，切片比数组更常见。

## 切片

切片是方便，灵活且功能强大的基于数组的装饰器。切片本身不拥有任何数据。它们只是对现有数组的引用。

#####  创建切片
一个元素类型为`T`的切片用`[ ]T`表示。

```golang
package main

import (  
    "fmt"
)

func main() {  
    a := [5]int{76, 77, 78, 79, 80}
    var b []int = a[1:4] //creates a slice from a[1] to a[3]
    fmt.Println(b)
}
```
[Run in playground](https://play.golang.org/p/Za6w5eubBB)

语法`a[start:end]`表示用数组`a`索引的`start`到`end - 1`来创建一个切片，所以上述程序第 9 行的`a[1:4]`创建了一个表示数组`a`从索引 1 到 3 的切片（左闭右开区间）。因此，切片`b`的值为`[77 78 79]`

让我们看下创建切片的另一种方式，

```golang
package main

import (  
    "fmt"
)

func main() {  
    c := []int{6, 7, 8} //creates and array and returns a slice reference
    fmt.Println(c)
}
```
[Run in playground](https://play.golang.org/p/_Z97MgXavA)

上述代码的第 9 行，`c := []int{6, 7, 8}`创建了一个 3 个整数的切片 c`。

##### 修改切片

切片不拥有自己的任何数据。它只是底层数组的表示，对切​​片所做的任何修改都将反映在底层数组中。

```golang
package main

import (  
    "fmt"
)

func main() {  
    darr := [...]int{57, 89, 90, 82, 100, 78, 67, 69, 59}
    dslice := darr[2:5]
    fmt.Println("array before",darr)
    for i := range dslice {
        dslice[i]++
    }
    fmt.Println("array after",darr) 
}
```
[Run in playground](https://play.golang.org/p/6FinudNf1k)

在上面程序的第 9 行中，我们从数组的索引 2, 3, 4 创建了`dslice`切片。 `for`循环将这些索引中的值递增 1。当我们在`for`循环之后打印数组时，我们可以看到对切片的更改影响到了数组。该程序的输出是
```plain
array before [57 89 90 82 100 78 67 69 59]  
array after [57 89 91 83 101 78 67 69 59]  
```

当许多切片共享相同的底层数组时，每个切片所做的更改将反映在数组中。

```golang
package main

import (  
    "fmt"
)

func main() {  
    numa := [3]int{78, 79 ,80}
    nums1 := numa[:] //creates a slice which contains all elements of the array
    nums2 := numa[:]
    fmt.Println("array before change 1",numa)
    nums1[0] = 100
    fmt.Println("array after modification to slice nums1", numa)
    nums2[1] = 101
    fmt.Println("array after modification to slice nums2", numa)
}
```
[Run in playground](https://play.golang.org/p/mdNi4cs854)

在第 9 行，`numa[:]`中缺少了起始值和结束值。`start`和`end`的默认值分别为`0`和`len(numa)`。切片`nums1`和`nums2`共享相同的数组。该程序的输出是
```plain
array before change 1 [78 79 80]  
array after modification to slice nums1 [100 79 80]  
array after modification to slice nums2 [100 101 80]  
```

从输出中可以清楚地看出，当切片共享同一个数组时，每个数组所做的修改都会反映在数组中。

## 切片的长度和容量
切片的长度是指切片中元素的数量。切片的容量是指从切片的起始索引值开始到数组末尾的元素数。

写个代码来加深理解，

```golang
package main

import (  
    "fmt"
)

func main() {  
    fruitarray := [...]string{"apple", "orange", "grape", "mango", "water melon", "pine apple", "chikoo"}
    fruitslice := fruitarray[1:3]
    fmt.Printf("length of slice %d capacity %d", len(fruitslice), cap(fruitslice)) //length of is 2 and capacity is 6
}
```
[Run in playground](https://play.golang.org/p/a1WOcdv827)

在上面的程序中，`fruitslice`是用`fruitarray`的索引值 1 和 2 创建的。因此，长度为 2。

`fruitarray`的长度为 7。`fruiteslice`从`fruitarray`的索引 1 开始创建。因此，`fruiteslice`的容量是从索引 1 开始的`fruitarray`中的元素，即从`orange`开始，该值为 6。因此，`fruiteslice`的容量为 6。该程序输出`length of slice 2 capacity 6.`

切片可以重新切片到他的最大容量。如果超过容量将会导致程序运行时抛出错误。

```golang
package main

import (  
    "fmt"
)

func main() {  
    fruitarray := [...]string{"apple", "orange", "grape", "mango", "water melon", "pine apple", "chikoo"}
    fruitslice := fruitarray[1:3]
    fmt.Printf("length of slice %d capacity %d\n", len(fruitslice), cap(fruitslice)) //length of is 2 and capacity is 6
    fruitslice = fruitslice[:cap(fruitslice)] //re-slicing furitslice till its capacity
    fmt.Println("After re-slicing length is",len(fruitslice), "and capacity is",cap(fruitslice))
}
```
[Run in playground](https://play.golang.org/p/GcNzOOGicu)

在第 11 行，`fruitslice`被重新切片到他的最大容量，程序将输出，
```plain
length of slice 2 capacity 6  
After re-slicing length is 6 and capacity is 6  
```

##### 使用 make 函数创建切片

`func make([]T, len, cap) []T` 通过传递类型，长度，容量来创建一个切片。容量参数是可选的，默认值是长度。

`make`函数创建一个数组并返回一个切片引用。

```golang
package main

import (  
    "fmt"
)

func main() {  
    i := make([]int, 5, 5)
    fmt.Println(i)
}
```
[Run in playground](https://play.golang.org/p/M4OqxzerxN)
使用`make`函数创建的切片默认会被零值填充，上述程序将输出`[0 0 0 0 0]`

##### 为切片增加元素
我们知道数组是固定长度的，并且它们的长度不能扩容。而切片是动态的，可以使用`append`函数将新元素增加到切片。`func append(s []T, x ...T) []T`是`append`函数的定义。

函数中的`x ...T`参数表示函数可以接受一个变长的参数`x`，这种类型的函数称为[变参函数]([https://golangbot.com/variadic-functions/](https://golangbot.com/variadic-functions/)
)。

但有一个问题可能会困扰你。切片的底层依赖了数组，而数组本身是固定长度，那么切片是怎么实现动态长度的呢？实现原理是，当新元 素添加到切片时，会创建一个新数组。现有数组的元素将复制到此新数组，并返回此新数组的新切片引用。新切片的容量现在是旧切片的两倍（译者注：有一个扩容算法，并不都是两倍），下面的代码将会让我更容易理解。

```golang
package main

import (  
    "fmt"
)

func main() {  
    cars := []string{"Ferrari", "Honda", "Ford"}
    fmt.Println("cars:", cars, "has old length", len(cars), "and capacity", cap(cars)) //capacity of cars is 3
    cars = append(cars, "Toyota")
    fmt.Println("cars:", cars, "has new length", len(cars), "and capacity", cap(cars)) //capacity of cars is doubled to 6
}
```

[Run in playground](https://play.golang.org/p/VUSXCOs1CF)

在上述程序中，`cars`的容量最初为 3。我们在第 8 行中为`cars`添加了一个新元素，并把`append(cars, "Toyota")`返回的切片赋值给`cars`。现在`cars`的容量增加了一倍，变成了 6。上述程序的输出是
```plain
cars: [Ferrari Honda Ford] has old length 3 and capacity 3  
cars: [Ferrari Honda Ford Toyota] has new length 4 and capacity 6  
```

切片的零值是`nil`，一个`nil`切片的长度和容量都为 0。能使用`append`函数给`nil`切片添加元素。

```golang
package main

import (  
    "fmt"
)

func main() {  
    var names []string //zero value of a slice is nil
    if names == nil {
        fmt.Println("slice is nil going to append")
        names = append(names, "John", "Sebastian", "Vinay")
        fmt.Println("names contents:",names)
    }
}
```
[Run in playground](https://play.golang.org/p/x_-4XAJHbM)

`names`是一个`nil`切片，我们给`names`添加了 3 个字符串，该程序输出，
```plain
slice is nil going to append  
names contents: [John Sebastian Vinay]  
```

也可以使用`...`操作把一个切片添加到另一个切片，可以在[变参函数]([https://golangbot.com/variadic-functions/](https://golangbot.com/variadic-functions/)
)这一章节学到更多关于该操作的内容。

```golang
package main

import (  
    "fmt"
)

func main() {  
    veggies := []string{"potatoes","tomatoes","brinjal"}
    fruits := []string{"oranges","apples"}
    food := append(veggies, fruits...)
    fmt.Println("food:",food)
}
```
[Run in playground](https://play.golang.org/p/UnHOH_u6HS)

在上述代码的第 10 行，`food`切片使用`veggies`切片添加`fruits`切片创建。程序的输出为，`food: [potatoes tomatoes brinjal oranges apples]`

##### 切片作为函数的参数
切片的内部结构可以被认为是结构类型。这就是像这样，
```golang
type slice struct {  
    Length        int
    Capacity      int
    ZerothElement *byte
}
```

切片包含长度，容量和指向数组的第 0 个元素的指针。当切片传递给函数时，即使它是按值传递的，指针变量也会引用相同的底层数组。因此，当切片作为参数传递给函数时，函数内部所做的更改也会在函数外部显示。让我们写一个程序来验证一下。

```golang
package main

import (  
    "fmt"
)

func subtactOne(numbers []int) {  
    for i := range numbers {
        numbers[i] -= 2
    }

}
func main() {  
    nos := []int{8, 7, 6}
    fmt.Println("slice before function call", nos)
    subtactOne(nos)                               //function modifies the slice
    fmt.Println("slice after function call", nos) //modifications are visible outside
}
```
[Run in playground](https://play.golang.org/p/IzqDihNifq)

上述程序的第 16 行中的函数调用将每个切片元素递减 2。当在函数调用之后打印切片时，这些修改是可见的。回想一下，这和数组不同，在数组中，对函数内部的数组所做的修改在函数外部是不可见的。上述程序的输出是，
```plain
slice before function call [8 7 6]  
slice after function call [6 5 4]  
```

##### 多维切片
跟数组一样，切片也支持多维

```golang
package main

import (  
    "fmt"
)


func main() {  
     pls := [][]string {
            {"C", "C++"},
            {"JavaScript"},
            {"Go", "Rust"},
            }
    for _, v1 := range pls {
        for _, v2 := range v1 {
            fmt.Printf("%s ", v2)
        }
        fmt.Printf("\n")
    }
}
```
[Run in playground](https://play.golang.org/p/--p1AvNGwN)

程序输出，
```plain
C C++  
JavaScript  
Go Rust  
```

##### 内存优化
切片引用了底层数组，只要切片在内存中，就不能对数组进行垃圾回收。在内存管理，这可能会引起关注。我们假设我们有一个非常大的数组，我们有兴趣只处理它的一小部分。此后，我们从该数组创建一个切片并开始处理切片。需要注意的一点是，由于切片引用了该数组，因此数组仍将在内存中。

解决此问题的一种方法是使用`copy`函数`func copy(dst, src []T) int`来获取该切片的副本。这样我们就可以使用新的切片，原始数组就可以被垃圾收集了。

```golang
package main

import (  
    "fmt"
)

func countries() []string {  
    countries := []string{"USA", "Singapore", "Germany", "India", "Australia"}
    neededCountries := countries[:len(countries)-2]
    countriesCpy := make([]string, len(neededCountries))
    copy(countriesCpy, neededCountries) //copies neededCountries to countriesCpy
    return countriesCpy
}
func main() {  
    countriesNeeded := countries()
    fmt.Println(countriesNeeded)
}
```
[Run in playground](https://play.golang.org/p/35ayYBhcDE)

在上述代码的第 9 行，`neededCountries := countries[:len(countries)-2]`创建了一个`countries`的切片。在第 11 行将`neededCountries`复制给了`countriesCpy`，随后函数返回。现在，`countries`的底层数组将会被垃圾回收，因为`neededCountries`不再被引用。


