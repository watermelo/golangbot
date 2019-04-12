> * 原文地址：[Part 27: Composition Instead of Inheritance - OOP in Go](https://golangbot.com/inheritance/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

Go 不支持继承，但它支持组合。组合的通用定义是“放在一起”。组合的一个例子是汽车。汽车由车轮，发动机和各种其他部件组成。

## 通过嵌入结构组合
Go 中的组合可以通过将一种结构类型嵌入到另一种结构类型中来实现。

博客文章是一个完美的组合示例。每篇博文都有标题，内容和作者信息。这可以使用组合完美地表示。在本教程的后续步骤中，我们将了解如何完成此操作。

让我们先创建一个`author`结构

```golang
package main

import (  
    "fmt"
)

type author struct {  
    firstName string
    lastName  string
    bio       string
}

func (a author) fullName() string {  
    return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}
```

在上面的代码片段中，我们创建了一个包含`firstName`，`lastName`和`bio`字段的`author`结构。我们还添加了一个方法`fullName()`，`author`作为接收者类型，将返回作者的全名。

下一步是创建`post`结构。

```golang
type post struct {  
    title     string
    content   string
    author
}

func (p post) details() {  
    fmt.Println("Title: ", p.title)
    fmt.Println("Content: ", p.content)
    fmt.Println("Author: ", p.author.fullName())
    fmt.Println("Bio: ", p.author.bio)
}
```
`post`结构具有字段`title`，`content`。它还嵌入一个匿名字段`author`。该字段表示`post`结构由`author`组成。现在`post`结构可以访问`author`结构的所有字段和方法。我们还在 post 结构中添加了`details()`方法，用于打印作者的`title`，`content`，`fullName`和`bio`。

每当一个结构嵌入另一个结构时，Go 为我们提供了访问嵌入字段的选项，就好像它们是外部结构的一部分一样。这意味着`p.author.fullName()`可以用`p.fullName()`替换。因此，`details()`方法可以重写如下，
```golang
func (p post) details() {  
    fmt.Println("Title: ", p.title)
    fmt.Println("Content: ", p.content)
    fmt.Println("Author: ", p.fullName())
    fmt.Println("Bio: ", p.bio)
}
```
现在我们已经准备好了`author`和`post`结构，让我们通过创建一个博客帖子来完成这个程序。
```golang
package main

import (  
    "fmt"
)

type author struct {  
    firstName string
    lastName  string
    bio       string
}

func (a author) fullName() string {  
    return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

type post struct {  
    title   string
    content string
    author
}

func (p post) details() {  
    fmt.Println("Title: ", p.title)
    fmt.Println("Content: ", p.content)
    fmt.Println("Author: ", p.fullName())
    fmt.Println("Bio: ", p.bio)
}

func main() {  
    author1 := author{
        "Naveen",
        "Ramanathan",
        "Golang Enthusiast",
    }
    post1 := post{
        "Inheritance in Go",
        "Go supports composition instead of inheritance",
        author1,
    }
    post1.details()
}
```
[Run in playground](https://play.golang.org/p/sskWaTpJgr)

上面程序中的主要功能是在第 31 行中创建一个新的作者，在第 36 行嵌入该作者然后创建一个新帖子。这个程序打印，
```plain
Title:  Inheritance in Go  
Content:  Go supports composition instead of inheritance  
Author:  Naveen Ramanathan  
Bio:  Golang Enthusiast  
```

## 嵌入结构切片
我们可以进一步采用这个例子，并使用一些博客帖子创建一个网站:)。

让我们先定义`website`结构。我们将在现有程序的基础上添加以下代码并运行它。
```golang
type website struct {  
        []post
}
func (w website) contents() {  
    fmt.Println("Contents of Website\n")
    for _, v := range w.posts {
        v.details()
        fmt.Println()
    }
}
```
在添加上面的代码后，运行时编译器会报以下错误，
```plain
main.go:31:9: syntax error: unexpected [, expecting field name or embedded type  
```
此错误指向结构`[]post`的嵌入切片。原因是不能匿名嵌入一个切片。字段名称是必需的。所以让我们修复这个错误。
```plain
type website struct {  
        posts []post
}
```
我已将字段名称`posts`添加到`[]post`的切片中。

现在让我们修改`main function`并为我们的新网站创建一些帖子。

修改后的完整程序如下，

```golang
package main

import (  
    "fmt"
)

type author struct {  
    firstName string
    lastName  string
    bio       string
}

func (a author) fullName() string {  
    return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

type post struct {  
    title   string
    content string
    author
}

func (p post) details() {  
    fmt.Println("Title: ", p.title)
    fmt.Println("Content: ", p.content)
    fmt.Println("Author: ", p.fullName())
    fmt.Println("Bio: ", p.bio)
}

type website struct {  
 posts []post
}
func (w website) contents() {  
    fmt.Println("Contents of Website\n")
    for _, v := range w.posts {
        v.details()
        fmt.Println()
    }
}

func main() {  
    author1 := author{
        "Naveen",
        "Ramanathan",
        "Golang Enthusiast",
    }
    post1 := post{
        "Inheritance in Go",
        "Go supports composition instead of inheritance",
        author1,
    }
    post2 := post{
        "Struct instead of Classes in Go",
        "Go does not support classes but methods can be added to structs",
        author1,
    }
    post3 := post{
        "Concurrency",
        "Go is a concurrent language and not a parallel one",
        author1,
    }
    w := website{
        posts: []post{post1, post2, post3},
    }
    w.contents()
}
```
[Run in playground](https://play.golang.org/p/gKaa0RbeAE)

在上面的`main function`中，我们创建了一个作者`author1`和三个帖子`post1`，`post2`和`post3`。最后我们在第 62 行创建了网站`w`。 通过嵌入这 3 个帖子并在下一行显示内容。

程序输出，
```plain
Contents of Website

Title:  Inheritance in Go  
Content:  Go supports composition instead of inheritance  
Author:  Naveen Ramanathan  
Bio:  Golang Enthusiast

Title:  Struct instead of Classes in Go  
Content:  Go does not support classes but methods can be added to structs  
Author:  Naveen Ramanathan  
Bio:  Golang Enthusiast

Title:  Concurrency  
Content:  Go is a concurrent language and not a parallel one  
Author:  Naveen Ramanathan  
Bio:  Golang Enthusiast  
```

