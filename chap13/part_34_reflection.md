> * 原文地址：[Part 34: Reflection](https://golangbot.com/reflection/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。


反射是 Go 的高级主题之一，我会尽量让它变得简单。

本教程包含以下部分。

- 什么是反射？
- 检查变量并找到其类型需要做什么？
- `reflect` 包
  - `reflect.Type` 和 `reflect.Value`
  - `reflect.Kind`
  - `NumField()` 和 `Field()` 方法
  - `Int()` 和 `String()` 方法
- 完整的程序
- 应该使用反射吗？

我们现在一个一个来讨论这些部分。

## 什么是反射
反射是为了程序在运行时检查其变量和值并找到其类型。你可能不明白这意味着什么，但没关系。在本教程结束时，您将清楚地了解反射，请跟紧我。

## 检查变量并找到其类型需要做什么

在学习反射时，任何人都会想到的第一个问题是，为什么我们需要检查变量并在运行时找到它的类型，因为我们的程序中的每个变量都由我们定义的，我们在编译时就知道它的类型。嗯，大部分时间都是如此，但并非总是如此。

我们用一个简单的程序来解释一下我的意思。
```golang
package main

import (  
    "fmt"
)

func main() {  
    i := 10
    fmt.Printf("%d %T", i, i)
}
```
[Run in playgound](https://play.golang.org/p/1oZzPCCG2Qw)

在上面的程序中，`i`的类型在编译时是已知的，我们在下一行打印它。这里没什么神奇的。

现在让我们理解在运行时知道变量类型的必要性。假设我们想编写一个简单的函数，它将结构作为参数，并使用它创建一个 SQL 插入查询。

看看下的代码，

```golang
package main

import (  
    "fmt"
)

type order struct {  
    ordId      int
    customerId int
}

func main() {  
    o := order{
        ordId:      1234,
        customerId: 567,
    }
    fmt.Println(o)
}
```
[Run in playground](https://play.golang.org/p/0wLQAVErHuH)

我们需要编写一个函数，它将上面程序中的结构 `o` 作为参数并返回以下 SQL 插入语句，
```plain
insert into order values(1234, 567)  
```

这个功能很容易实现。让我们现在就搞。

```golang
package main

import (  
    "fmt"
)

type order struct {  
    ordId      int
    customerId int
}

func createQuery(o order) string {  
    i := fmt.Sprintf("insert into order values(%d, %d)", o.ordId, o.customerId)
    return i
}

func main() {  
    o := order{
        ordId:      1234,
        customerId: 567,
    }
    fmt.Println(createQuery(o))
}
```
[Run in playground](https://play.golang.org/p/jhz4VHKIlQ5)

第 12 行中的`createQuery`函数，使用`o`的`ordId`和`customerId`字段创建插入查询。该程序将输出，
```plain
insert into order values(1234, 567)  
```

现在让我们的`createQuery`进入下一个级别。如果我们想要概括我们的`createQuery`并使其适用于任何结构，该怎么办？让我解释一下我使用程序的意思。

```golang
package main

type order struct {  
    ordId      int
    customerId int
}

type employee struct {  
    name string
    id int
    address string
    salary int
    country string
}

func createQuery(q interface{}) string {  
}

func main() {

}
```

我们的目标是完成以上程序第 16 行的`createQuery`函数，以便它将任何结构作为参数，并基于结构字段创建插入查询。

例如，如果我们传递下面的结构，

```plain
o := order {  
    ordId: 1234,
    customerId: 567
}
```
我们的`createQuery`函数应该返回，
```plain
insert into order values (1234, 567) 
```

同样的我们传下面的结构，
```plain
 e := employee {
        name: "Naveen",
        id: 565,
        address: "Science Park Road, Singapore",
        salary: 90000,
        country: "Singapore",
    }
```
应该返回，
```plain
insert into employee values("Naveen", 565, "Science Park Road, Singapore", 90000, "Singapore")  
```

由于`createQuery`函数应该与任何结构一起使用，因此它将`interface{}`作为参数。为简单起见，我们只处理包含`string`和`int`类型字段的结构，但这可以扩展为任何类型。

`createQuery`函数应该适用于任何结构。编写此函数的唯一方法是检查在运行时传递给它的结构参数的类型，找到它的字段然后创建查询。这是应用反射的地方。在本教程的后续步骤中，我们将学习如何使用`reflect`包实现此目的。

## `reflect` 包

`reflect` 包在 Go 中实现了运行时的反射。`reflect` 包有助于识别底层具体类型和`interface {}`变量的值。这正是我们所需要的。`createQuery`函数采用`interface {}`参数，而创建查询需要`interface {}`参数的具体类型和值。这正是反射包有用的地方。

在编写我们的通用查询生成器程序之前，我们首先需要知道`reflect` 包中的一些类型和方法。让我们逐一看看它们。

## `reflect.Type` 和 `reflect.Value`

`interface {}`的具体类型由`reflect.Type`表示，底层值由`reflect.Value`表示。有两个函数`reflect.TypeOf()`和`reflect.ValueOf()`，它们分别返回`reflect.Type`和`reflect.Value`。这两种类型是创建查询生成器的基础。让我们写一个简单的例子来理解这两种类型。

```golang
package main

import (  
    "fmt"
    "reflect"
)

type order struct {  
    ordId      int
    customerId int
}

func createQuery(q interface{}) {  
    t := reflect.TypeOf(q)
    v := reflect.ValueOf(q)
    fmt.Println("Type ", t)
    fmt.Println("Value ", v)
}

func main() {  
    o := order{
        ordId:      456,
        customerId: 56,
    }
    createQuery(o)

}
```
[Run in playground](https://play.golang.org/p/81BS-bEfbCg)

在上面的程序中，第 13 行的`createQuery`函数以`interface {}`作为参数。第 14 行的函数`reflect.TypeOf`将`interface {}`作为参数，并返回包含传递的`interface {}`参数的具体类型的`reflect.Type`。类似，第 15 行中的·reflect.ValueOf·函数也将`interface {}`作为参数并返回`reflect.Value`，其中包含传递的`interface {}`参数的基础值。

上述程序打印，
```plain
Type  main.order  
Value  {456 56}  
```
从输出中，我们可以看到程序打印了接口的具体类型和值。

## `reflect.Kind`

反射包中有一个更重要的类型叫`Kind`。

反射包中的`Kind`和 `Type`可能看起来相似，但它们之间存在差异，这将从下面的程序中清楚地看出。

```golang
package main

import (  
    "fmt"
    "reflect"
)

type order struct {  
    ordId      int
    customerId int
}

func createQuery(q interface{}) {  
    t := reflect.TypeOf(q)
    k := t.Kind()
    fmt.Println("Type ", t)
    fmt.Println("Kind ", k)
}

func main() {  
    o := order{
        ordId:      456,
        customerId: 56,
    }
    createQuery(o)
}
```
[Run in playground](https://play.golang.org/p/Xw3JIzCm54T)

上述程序输出，
```plain
Type  main.order  
Kind  struct  
```

我想你现在会清楚两者之间的差异。 `Type`表示`interface {}`的实际类型，在这个例子中，它是`main.Order`。而`Kind`表示类型的特定种类。在这个例子中，它是一个`struct`。

## `NumField()`和`Field()`方法
`NumField()`方法返回结构中的字段数，`Field(i int)`方法返回第`i`个字段的`reflect.Value`。
```golang
package main

import (  
    "fmt"
    "reflect"
)

type order struct {  
    ordId      int
    customerId int
}

func createQuery(q interface{}) {  
    if reflect.ValueOf(q).Kind() == reflect.Struct {
        v := reflect.ValueOf(q)
        fmt.Println("Number of fields", v.NumField())
        for i := 0; i < v.NumField(); i++ {
            fmt.Printf("Field:%d type:%T value:%v\n", i, v.Field(i), v.Field(i))
        }
    }

}
func main() {  
    o := order{
        ordId:      456,
        customerId: 56,
    }
    createQuery(o)
}
```
[Run in playground](https://play.golang.org/p/FBHfJfuTaEe)

在上面的程序中的第 14 行，我们首先检查`q`的`Kind`是否是结构，因为`NumField`方法仅适用于结构。该代码的其余部分比较好懂，该程序输出，
```plain
Number of fields 2  
Field:0 type:reflect.Value value:456  
Field:1 type:reflect.Value value:56  
```

## `Int()`和 `String()`方法

`Int`和`String`方法有助于将`reflect.Value`分别提取为`int64`和`string`类型。

```golang
package main

import (  
    "fmt"
    "reflect"
)

func main() {  
    a := 56
    x := reflect.ValueOf(a).Int()
    fmt.Printf("type:%T value:%v\n", x, x)
    b := "Naveen"
    y := reflect.ValueOf(b).String()
    fmt.Printf("type:%T value:%v\n", y, y)

}
```
[Run in playground](https://play.golang.org/p/UIllrLVoGwI)

在上面的程序中的第 10 行，我们将`reflect.Value`提取为`int64`，在第 13 行，我们把它作为`string`提取出来。这个程序打印，
```plain
type:int64 value:56  
type:string value:Naveen  
```

## 完整的程序

现在我们有足够的知识来完成我们的查询生成器，那就让我们继续吧。

```golang
package main

import (  
    "fmt"
    "reflect"
)

type order struct {  
    ordId      int
    customerId int
}

type employee struct {  
    name    string
    id      int
    address string
    salary  int
    country string
}

func createQuery(q interface{}) {  
    if reflect.ValueOf(q).Kind() == reflect.Struct {
        t := reflect.TypeOf(q).Name()
        query := fmt.Sprintf("insert into %s values(", t)
        v := reflect.ValueOf(q)
        for i := 0; i < v.NumField(); i++ {
            switch v.Field(i).Kind() {
            case reflect.Int:
                if i == 0 {
                    query = fmt.Sprintf("%s%d", query, v.Field(i).Int())
                } else {
                    query = fmt.Sprintf("%s, %d", query, v.Field(i).Int())
                }
            case reflect.String:
                if i == 0 {
                    query = fmt.Sprintf("%s\"%s\"", query, v.Field(i).String())
                } else {
                    query = fmt.Sprintf("%s, \"%s\"", query, v.Field(i).String())
                }
            default:
                fmt.Println("Unsupported type")
                return
            }
        }
        query = fmt.Sprintf("%s)", query)
        fmt.Println(query)
        return

    }
    fmt.Println("unsupported type")
}

func main() {  
    o := order{
        ordId:      456,
        customerId: 56,
    }
    createQuery(o)

    e := employee{
        name:    "Naveen",
        id:      565,
        address: "Coimbatore",
        salary:  90000,
        country: "India",
    }
    createQuery(e)
    i := 90
    createQuery(i)

}
```
[Run in playground](https://play.golang.org/p/82Bi4RU5c7W)

在第 22 行，我们首先检查传递的参数是否为结构。然后我们使用`Name()`方法从`reflect.Type`获取结构的名称。在下一行中，我们使用`t`并开始创建查询。

第 28 行，`case`语句检查当前字段是否为`reflect.Int`，如果是的话，我们使用`Int()`方法将该字段的值提取为`int64`。 `if else`语句用于处理边缘情况，请添加日志以了解为何需要它。类似的逻辑用于提取`string`。

我们还添加了一些检查，以防止在将不支持的类型传递给`createQuery`函数时导致程序崩溃。该程序的其他代码也比较好懂，建议在适当的位置添加日志并检查其输出以更好地理解该程序。

程序输出，
```plain
insert into order values(456, 56)  
insert into employee values("Naveen", 565, "Coimbatore", 90000, "India")  
unsupported type 
```

留下一个练习给读者，如果我们要将字段名称添加到输出查询中，该怎么修改？请尝试更改程序以打印如下格式的查询，
```plain
insert into order(ordId, customerId) values(456, 56)  
```

## 应该使用反射吗

我们展示了一个反射的用途。现在有一个实际的问题，你应该使用反射吗？我想引用 Rob Pike 关于使用反射的回答来解释这个问题。

*Clear is better than clever. Reflection is never clear.* -- 清晰比聪明更好，但是反射不清晰

在 Go 中，反射是一个非常强大和先进的概念，应该谨慎使用。使用反射编写清晰且可维护的代码非常困难。应尽可能避免使用，并且只有在绝对必要时才应使用。

如果喜欢我的教程，可以向我[捐赠]([https://golangbot.com/support-the-content/](https://golangbot.com/support-the-content/)
)。您的捐款将帮助我创建更多精彩的教程。





