> * 原文地址：[Part 3: Variables](https://golangbot.com/variables/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

This is the third tutorial in our [Golang tutorial series](https://golangbot.com/learn-golang-series/) and it deals with variables in golang.

You can read the [Golang tutorial part 2: Hello World](https://golangbot.com/hello-world/) to learn about configuring golang and running the hello world program.

### What is a variable

Variable is the name given to a memory location to store a value of a specific type. There are various syntaxes to declare variables in go.

### Declaring a single variable

**var name type** is the syntax to declare a single variable.

```golang
package main

import "fmt"

func main() {  
    var age int // variable declaration
    fmt.Println("my age is", age)
}
```

[Run in playground](https://play.golang.org/p/XrveIxw_YI)

The statement `var age int` declares a variable named age of type `int`. We have not assigned any value for the variable. If a variable is not assigned any value, go automatically initialises it with the **zero value** of the variable's type. In this case age is assigned the value 0. If you run this program, you can see the following output

```    
my age is 0  
```
    

A variable can be assigned to any value of its type. In the above program age can be assigned any integer value.

```golang
package main

import "fmt"

func main() {  
    var age int // variable declaration
    fmt.Println("my age is ", age)
    age = 29 //assignment
    fmt.Println("my age is", age)
    age = 54 //assignment
    fmt.Println("my new age is", age)
}
```

[Run in playground](https://play.golang.org/p/z4nKMjBxLx)

The above program will produce the following output.

```
my age is  0  
my age is 29  
my new age is 54  
```
    

### Declaring a variable with initial value

A variable can also be given a initial value when it is declared.

**var name type = initialvalue** is the syntax to declare a variable with initial value.

```golang
package main

import "fmt"

func main() {  
    var age int = 29 // variable declaration with initial value

    fmt.Println("my age is", age)
}
```
    

[Run in playground](https://play.golang.org/p/TFfpzsrchh)

In the above program, age is variable of type int and has initial value 29. If you run the above program, you can see the following output. It shows that age has been initialised with the value 29.

```
my age is 29  
```

### Type inference

If a variable has an initial value, Go will automatically be able to infer the type of that variable using that initial value. Hence if a variable has an initial value, the _type_ in the variable declaration can be omitted.

If the variable is declared using the syntax **var name = initialvalue**, Go will automatically infer the type of that variable from the initial value.

In the following example, you can see that the type _int_ of the variable _age_ has been removed in line no. 6. Since the variable has a initial value of 29, go can infer that it is of type int.

```golang
package main

import "fmt"

func main() {  
    var age = 29 // type will be inferred

    fmt.Println("my age is", age)
}
```

[Run in playground](https://play.golang.org/p/FgNbfL3WIt)

### Multiple variable declaration

Multiple variables can be declared in a single statement.

**var name1, name2 type = initialvalue1, initialvalue2** is the syntax for multiple variable declaration.

```golang
package main

import "fmt"

func main() {  
    var width, height int = 100, 50 //declaring multiple variables

    fmt.Println("width is", width, "height is", height)
}
```    

[Run in playground](https://play.golang.org/p/4aOQyt55ah)

The _type_ can be omitted if the variables have initial value. The program below declares multiple variables using type inference.

```golang
package main

import "fmt"

func main() {  
    var width, height = 100, 50 //"int" is dropped

    fmt.Println("width is", width, "height is", height)
}
```

[Run in playground](https://play.golang.org/p/sErofTJ6g-)

The above program will print `width is 100 height is 50` as the output.

As you would have probably guessed by now, if the initial value is not specified for width and height, they will have 0 assigned as their initial value.

```golang
    package main
    
    import "fmt"
    
    func main() {  
        var width, height int
        fmt.Println("width is", width, "height is", height)
        width = 100
        height = 50
        fmt.Println("new width is", width, "new height is ", height)
    }
```    

[Run in playground](https://play.golang.org/p/DM00pcBbsu)

The above program will print

```
width is 0 height is 0  
new width is 100 new height is  50  
```

There might be cases where we would want to declare variables belonging to different types in a single statement. The syntax for doing that is

```golang
var (  
      name1 = initialvalue1,
      name2 = initialvalue2
)
```

The following program uses the above syntax to declare variables of different types.

```golang
package main

import "fmt"

func main() {  
    var (
        name   = "naveen"
        age    = 29
        height int
    )
    fmt.Println("my name is", name, ", age is", age, "and height is", height)
}
```

[Run in playground](https://play.golang.org/p/7pkp74h_9L)

Here we declare a variable **name of type string, age and height of type int**. (We will discuss about the various types available in golang in the next tutorial). Running the above program will produce the output `my name is naveen , age is 29 and height is 0`

### Short hand declaration

Go also provides another concise way for declaring variables. This is known as short hand declaration and it uses **:=** operator.

**name := initialvalue** is the short hand syntax to declare a variable.

```golang
package main

import "fmt"

func main() {  
    name, age := "naveen", 29 //short hand declaration

    fmt.Println("my name is", name, "age is", age)
}
```

[Run in playground](https://play.golang.org/p/ctqgw4w6kx)

If you run the above program, you can see `my name is naveen age is 29` as the output.

Short hand declaration requires initial values for all variables in the left hand side of the assignment. The following program will thrown an error `cannot assign 1 values to 2 variables`. This is because **age has not been assigned a value.**

```golang
package main

import "fmt"

func main() {  
    name, age := "naveen" //error

    fmt.Println("my name is", name, "age is", age)
}
``` 

[Run in playground](https://play.golang.org/p/wZd2HmDvqw)

Short hand syntax can only be used when at least one of the variables in the left side of **:=** is newly declared. Consider the following program,

```golang
package main

import "fmt"

func main() {  
    a, b := 20, 30 // declare variables a and b
    fmt.Println("a is", a, "b is", b)
    b, c := 40, 50 // b is already declared but c is new
    fmt.Println("b is", b, "c is", c)
    b, c = 80, 90 // assign new values to already declared variables b and c
    fmt.Println("changed b is", b, "c is", c)
}
```

[Run in playground](https://play.golang.org/p/MSUYR8vazB)

In the above program, in line no. 8, **b** has already been declared but **c** is newly declared and hence it works and outputs

```
a is 20 b is 30  
b is 40 c is 50  
changed b is 80 c is 90  
```

Whereas if we run the program below,

```golang
package main

import "fmt"

func main() {  
    a, b := 20, 30 //a and b declared
    fmt.Println("a is", a, "b is", b)
    a, b := 40, 50 //error, no new variables
}
```

[Run in playground](https://play.golang.org/p/EYTtRnlDu3)

it will throw error `no new variables on left side of :=` This is because both the variables **a** and **b** have already been declared and there are no new variables in the left side of **:=**

Variables can also be assigned values which are computed during run time. Consider the following program,

```golang
package main

import (  
    "fmt"
    "math"
)

func main() {  
    a, b := 145.8, 543.8
    c := math.Min(a, b)
    fmt.Println("minimum value is ", c)
}
```    

[Run in playground](https://play.golang.org/p/7XojAtrpH9)

In the above program, the value of c is calculated during run time and it's the minimum of a and b. The program above will print

```
    minimum value is  145.8  
```  

Since Go is strongly typed, variables declared as belonging to one type cannot be assigned a value of another type. The following program will throw an error `cannot use "naveen" (type string) as type int in assignment` because age is declared as type int and we are trying to assign a string value to it.

```golang
package main

func main() {  
    age := 29      // age is int
    age = "naveen" // error since we are trying to assign a string to a variable of type int
}
```    

[Run in playground](https://play.golang.org/p/K5rz4gxjPj)

Thanks for reading. Please post your feedback and queries in the comments section.

**Next tutorial - [Types](https://golangbot.com/types/)**