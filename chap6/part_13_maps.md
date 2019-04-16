> * 原文地址：[Part 13: Maps](https://golangbot.com/maps/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

## 什么是 map

map 是 Go 中的内置类型，它将值与键相关联。可以使用相应的键查找该值。

## 怎么创建 map
可以指定键和值的类型，然后通过`make`函数来创建一个 map，语法为`make(map[type of key]type of value)`

```golang
personSalary := make(map[string]int) 
```

上面的代码创建了一个叫`personSalary`的 map，它的键的类型为`string`，值的类型为`int`。

map 的零值为`nil`，如果你尝试给空 map 增加元素，运行时将会触发 panic，因此，必须使用`make`函数初始化 map。

```golang
package main

import (  
    "fmt"
)

func main() {  
    var personSalary map[string]int
    if personSalary == nil {
        fmt.Println("map is nil. Going to make one.")
        personSalary = make(map[string]int)
    }
}
```
[Run in playground](https://play.golang.org/p/IwJnXMGc1M)

在上述代码中，`personSalary`是空的，所以必须用 make 函数初始化。该代码输出`map is nil. Going to make one.`

## 给 map 增加元素

给 map 增加元素的语法和数组一样，下面的代码将给`personSalary`增加新的元素。

```golang
package main

import (  
    "fmt"
)

func main() {  
    personSalary := make(map[string]int)
    personSalary["steve"] = 12000
    personSalary["jamie"] = 15000
    personSalary["mike"] = 9000
    fmt.Println("personSalary map contents:", personSalary)
}
```

[Run in playground](https://play.golang.org/p/V1lnQ4Igw1)

上面的程序将会输出，`personSalary map contents: map[steve:12000 jamie:15000 mike:9000]`

map 也可以在初始化的时候添加元素，

```golang
package main

import (  
    "fmt"
)

func main() {  
    personSalary := map[string]int {
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    fmt.Println("personSalary map contents:", personSalary)
}
```

[Run in playground](https://play.golang.org/p/nlH_ADhO9f)

上面的程序声明了`personSalary`，并在声明过程中为它添加了两个元素。之后又添加了一个键为`mike`的元素。该程序输出

```plain
personSalary map contents: map[steve:12000 jamie:15000 mike:9000]  
```

键的类型并不仅仅是`string`。所有可比较的类型，如`boolean`，`integer`，`float`，`complex`，`string`，...也可以是键。如果你想了解有关类似类型的更多信息，请访问http://golang.org/ref/spec#Comparison_operators

## 访问 map 的元素

现在我们已经向 map 添加了一些元素，让我们学习如何访问它们。 `map[key]`是访问 map 元素的语法。

```golang
 
package main

import (  
    "fmt"
)

func main() {  
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    employee := "jamie"
    fmt.Println("Salary of", employee, "is", personSalary[employee])
}
```

[Run in playground](https://play.golang.org/p/-TSBac7F1v)

上述程序非常简单。员工 Jamie 的工资被找到并打印出来。程序输出`Salary of jamie is 15000.`

如果元素不存在会发生什么？map 将返回该元素类型的零值。在`personSalary`的例子中，如果我们尝试访问当前不存在的元素，则返回`int`的零值 0。

```golang
package main

import (  
    "fmt"
)

func main() {  
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    employee := "jamie"
    fmt.Println("Salary of", employee, "is", personSalary[employee])
    fmt.Println("Salary of joe is", personSalary["joe"])
}
```
[Run in playground](https://play.golang.org/p/EhUJhIkYJU)

上面的程序将输出，
```plain
Salary of jamie is 15000  
Salary of joe is 0  
```

上面的程序返回了 joe 的工资为 0，我们并没有得到 joe 不存在于`personSalary`中的信息。

如果我们想知道 map 中是否存在某个键，该怎么办？

```golang
value, ok := map[key] 
```

上面的语法就是找出 map 中是否存在某个键，如果`ok`是`true`的话，那就说明键存在，并且值为`value`。否则`ok`为`false`，`value`为空。

```golang
package main

import (  
    "fmt"
)

func main() {  
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    newEmp := "joe"
    value, ok := personSalary[newEmp]
    if ok == true {
        fmt.Println("Salary of", newEmp, "is", value)
    } else {
        fmt.Println(newEmp,"not found")
    }

}
```

[Run in playground](https://play.golang.org/p/q8fL6MeVZs)


上述程序的第 15 行，由于 joe 不存在，`ok`为`false`。因此程序输出
```plain
joe not found
```

使用`range for`可以迭代所有的 map 元素。

```golang
package main

import (  
    "fmt"
)

func main() {  
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    fmt.Println("All items of a map")
    for key, value := range personSalary {
        fmt.Printf("personSalary[%s] = %d\n", key, value)
    }

}
```

[Run in playground](https://play.golang.org/p/gq9ZOKsI9b)

上述程序输出，
```plain
All items of a map  
personSalary[mike] = 9000  
personSalary[steve] = 12000  
personSalary[jamie] = 15000  
```

一个重要的事实是，使用`range for`迭代获取元素的顺序，并不能保证结果每次相同。

## 删除元素

从 map 中删除一个 key 的语法为[`delete(map, key)`](https://golang.org/pkg/builtin/#delete)，删除函数不返回任何值。

```golang
package main

import (  
    "fmt"
)

func main() {  
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    fmt.Println("map before deletion", personSalary)
    delete(personSalary, "steve")
    fmt.Println("map after deletion", personSalary)

}
```
[Run in playground](https://play.golang.org/p/oGcBbRY3wky)


上述程序删除 key 为 steve 的元素，程序输出
```plain
map before deletion map[steve:12000 jamie:15000 mike:9000]  
map after deletion map[mike:9000 jamie:15000]  
```

## map 的长度
可以使用`len`函数确定 map 的长度。

```golang
package main

import (  
    "fmt"
)

func main() {  
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    fmt.Println("length is", len(personSalary))

}
```
[Run in playground](https://play.golang.org/p/8O1WnKUuDP)

上述程序使用`len(personSalary)`计算 map 的长度，输出，`length is 3`

## map 是指针传递
和 slice 一样，map 也是指针传递的。将 map 赋值给新变量时，它们都指向相同的内部数据结构。因此修改一个会影响另外一个。

```golang
package main

import (  
    "fmt"
)

func main() {  
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    fmt.Println("Original person salary", personSalary)
    newPersonSalary := personSalary
    newPersonSalary["mike"] = 18000
    fmt.Println("Person salary changed", personSalary)

}
```
[Run in playground](https://play.golang.org/p/OGFl3addq1)


在上述代码的第 14 行，`personSalary`赋值给`newPersonSalary`。在下一行，mike 的工资被修改为 18000，`personSalary`中的薪水也会变成 18000。程序输出，

```plain
Original person salary map[steve:12000 jamie:15000 mike:9000]  
Person salary changed map[steve:12000 jamie:15000 mike:18000]  
```

类似的，如果将 map 作为函数的接收者。当对函数内的 map 进行更改时，调用者将可以看到更改。

## map 的等值比较

map 的等值比较不能使用`==`操作符，`==`操作符仅仅能用来检查该 map 是否为`nil`。

```golang
package main

func main() {  
    map1 := map[string]int{
        "one": 1,
        "two": 2,
    }

    map2 := map1

    if map1 == map2 {
    }
}
```

[Run in playground](https://play.golang.org/p/MALqDyWkcT)


上述的代码将会抛出编译错误，`invalid operation: map1 == map2 (map can only be compared to nil).`

检查两个 map 是否相等的方法是逐个比较每个元素是否一样。为此编写一个函数是个不错的方法:)