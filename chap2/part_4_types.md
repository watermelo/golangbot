> * 原文地址：[Part 4: Types](https://golangbot.com/types/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

This is the tutorial number 4 in our [Golang tutorial series](https://golangbot.com/learn-golang-series/).

Please read [Golang tutorial part 3: Variables](https://golangbot.com/variables/) of this series to learn about variables.

The following are the basic types available in go

*   bool
*   Numeric Types
    *   int8, int16, int32, int64, int
    *   uint8, uint16, uint32, uint64, uint
    *   float32, float64
    *   complex64, complex128
    *   byte
    *   rune
*   string

### bool

A bool type represents a boolean and is either _true_ or _false_.

    package main
    
    import "fmt"
    
    func main() {  
        a := true
        b := false
        fmt.Println("a:", a, "b:", b)
        c := a && b
        fmt.Println("c:", c)
        d := a || b
        fmt.Println("d:", d)
    }
    

[Run in playground](https://play.golang.org/p/v_W3HQ0MdY)

In the program above, a is assigned to true and b is assigned a false value.

c is assigned the value of a && b. _The && operator returns true only when both a and b are true_. So in this case c is false.

_The || operator returns true when either a or b is true_. In this case d is assigned to true since a is true. We will get the following output for this program.

    a: true b: false  
    c: false  
    d: true  
    

### Signed integers

**int8:** represents 8 bit signed integers  
**size:** 8 bits  
**range:** -128 to 127

**int16:** represents 16 bit signed integers  
**size:** 16 bits  
**range:** -32768 to 32767

**int32:** represents 32 bit signed integers  
**size:** 32 bits  
**range:** -2147483648 to 2147483647

**int64:** represents 64 bit signed integers  
**size:** 64 bits  
**range:** -9223372036854775808 to 9223372036854775807

**int:** represents 32 or 64 bit integers depending on the underlying platform. You should generally be using _int_ to represent integers unless there is a need to use a specific sized integer.  
**size:** 32 bits in 32 bit systems and 64 bit in 64 bit systems.  
**range:** -2147483648 to 2147483647 in 32 bit systems and -9223372036854775808 to 9223372036854775807 in 64 bit systems

    package main
    
    import "fmt"
    
    func main() {  
        var a int = 89
        b := 95
        fmt.Println("value of a is", a, "and b is", b)
    }
    

[Run in playground](https://play.golang.org/p/NyDPsjkma3)

The above program will output `value of a is 89 and b is 95`

In the above program _a is of type int_ and _the type of b is inferred from the value assigned to it (95)._ As we have stated above, the size of _int is 32 bit in 32 bit systems and 64 bit in 64 bit systems_. Lets go ahead and verify this claim.

The type of a variable can be printed using **%T** format specifier in Printf method. Go has a package [unsafe](https://golang.org/pkg/unsafe/) which has a [Sizeof](https://golang.org/pkg/unsafe/#Sizeof) function which returns in bytes the size of the variable passed to it. _unsafe_ package should be used with care as the code using it might have portability issues, but for the purposes of this tutorial we can use it.

The following program outputs the type and size of both variables a and b. `%T` is the format specifier to print the type and `%d` is used to print the size.

    package main
    
    import (  
        "fmt"
        "unsafe"
    )
    
    func main() {  
        var a int = 89
        b := 95
        fmt.Println("value of a is", a, "and b is", b)
        fmt.Printf("type of a is %T, size of a is %d", a, unsafe.Sizeof(a)) //type and size of a
        fmt.Printf("\ntype of b is %T, size of b is %d", b, unsafe.Sizeof(b)) //type and size of b
    }
    

[Run in playground](https://play.golang.org/p/mFsmjVk5oc)

The above program will produce the output

    value of a is 89 and b is 95  
    type of a is int, size of a is 4  
    type of b is int, size of b is 4  
    

We can infer from the above output that a and b are of type _int_ and they are _32 bit sized(4 bytes)_. The output will vary if you run the above program on a 64 bit system. In a 64 bit system, a and b occupy 64 bits (8 bytes).

### Unsigned integers

**uint8:** represents 8 bit unsigned integers  
**size:** 8 bits  
**range:** 0 to 255

**uint16:** represents 16 bit unsigned integers  
**size:** 16 bits  
**range:** 0 to 65535

**uint32:** represents 32 bit unsigned integers  
**size:** 32 bits  
**range:** 0 to 4294967295

**uint64:** represents 64 bit unsigned integers  
**size:** 64 bits  
**range:** 0 to 18446744073709551615

**uint :** represents 32 or 64 bit unsigned integers depending on the underlying platform.  
**size :** 32 bits in 32 bit systems and 64 bits in 64 bit systems.  
**range :** 0 to 4294967295 in 32 bit systems and 0 to 18446744073709551615 in 64 bit systems

### Floating point types

**float32:** 32 bit floating point numbers  
**float64:** 64 bit floating point numbers

The following is a simple program to illustrate integer and floating point types

    package main
    
    import (  
        "fmt"
    )
    
    func main() {  
        a, b := 5.67, 8.97
        fmt.Printf("type of a %T b %T\n", a, b)
        sum := a + b
        diff := a - b
        fmt.Println("sum", sum, "diff", diff)
    
        no1, no2 := 56, 89
        fmt.Println("sum", no1+no2, "diff", no1-no2)
    }
    

[Run in playground](https://play.golang.org/p/upwUCprT-j)

The type of a and b is inferred from the value assigned to them. In this case a and b are of type float64.(float64 is the default type for floating point values). We add a and b and assign it to a variable sum. We subtract b from a and assign it to diff. Then the sum and diff is printed. Similar computation is done with no1 and no2. The above program will print

    type of a float64 b float64  
    sum 14.64 diff -3.3000000000000007  
    sum 145 diff -33  
    

### Complex types

**complex64:** complex numbers which have float32 real and imaginary parts  
**complex128:** complex numbers with float64 real and imaginary parts

The builtin function **[complex](https://golang.org/pkg/builtin/#complex)** is used to construct a complex number with real and imaginary parts. The complex function has the following definition

    func complex(r, i FloatType) ComplexType  
    

It takes a real and imaginary part as parameter and returns a complex type. _Both the real and imaginary parts should be of the same type. ie either float32 or float64. If both the real and imaginary parts are float32, this function returns a complex value of type complex64. If both the real and imaginary parts are of type float64, this function returns a complex value of type complex128_

Complex numbers can also be created using the shorthand syntax

    c := 6 + 7i  
    

Lets write a small program to understand complex numbers.

    package main
    
    import (  
        "fmt"
    )
    
    func main() {  
        c1 := complex(5, 7)
        c2 := 8 + 27i
        cadd := c1 + c2
        fmt.Println("sum:", cadd)
        cmul := c1 * c2
        fmt.Println("product:", cmul)
    }
    

[Run in playground](https://play.golang.org/p/kEz1uKCdKs)

In the above program, c1 and c2 are two complex numbers. c1 has 5 as real part and 7 as the imaginary part. c2 has real part 8 and imaginary part 27. `cadd` is assigned the sum of c1 and c2 and `cmul` is assigned the product of c1 and c2. This program will output

    sum: (13+34i)  
    product: (-149+191i)  
    

### Other numeric types

**byte** is an alias of uint8  
**rune** is an alias of int32

We will discuss bytes and runes in more detail when we learn about strings.

### string type

Strings are a collection of bytes in golang. It's alright if this definition doesn't make any sense. For now we can assume a string to be a collection of characters. We will learn about strings in detail in a separate tutorial.

Lets write a program using strings.

    package main
    
    import (  
        "fmt"
    )
    
    func main() {  
        first := "Naveen"
        last := "Ramanathan"
        name := first +" "+ last
        fmt.Println("My name is",name)
    }
    

[Run in playground](https://play.golang.org/p/CI6phwSVel)

In the above program, _first_ is assigned the string _"Naveen"_, _last_ is assigned the string "Ramanathan". Strings can be concatenated using the + operator. _name_ is assigned the value of _first_ concatenated to a _space_ followed by _last_. The above program will print `My name is Naveen Ramanathan` as the output.

There are some more operations that can performed on strings. We will look at those in a separate tutorial.

### Type Conversion

Go is very strict about explicit typing. There is no automatic type promotion or conversion. Lets look at what this means with an example.

    package main
    
    import (  
        "fmt"
    )
    
    func main() {  
        i := 55      //int
        j := 67.8    //float64
        sum := i + j //int + float64 not allowed
        fmt.Println(sum)
    }
    

[Run in playground](https://play.golang.org/p/m-TMRpmmnm)

The above code is perfectly legal in C language. But in the case of go, this wont work. i is of type int and j is of type float64. We are trying to add 2 numbers of different types which is not allowed. When you run the program, you will get `main.go:10: invalid operation: i + j (mismatched types int and float64)`

To fix the error, both _i_ and _j_ should be of the same type. Let's convert _j_ to int. _T(v) is the syntax to convert a value v to type T_

    package main
    
    import (  
        "fmt"
    )
    
    func main() {  
        i := 55      //int
        j := 67.8    //float64
        sum := i + int(j) //j is converted to int
        fmt.Println(sum)
    }
    

[Run in playground](https://play.golang.org/p/mweu3n3jMy)

Now when you run the above program, you can see `122` as the output.

The same is the case with assignment. Explicit type conversion is required to assign a variable of one type to another. This is explained in the following program.

    package main
    
    import (  
        "fmt"
    )
    
    func main() {  
        i := 10
        var j float64 = float64(i) //this statement will not work without explicit conversion
        fmt.Println("j", j)
    }
    

[Run in playground](https://play.golang.org/p/Y2uSYYr46c)

In line no. 9, i is converted to float64 and then assigned to j. When you try to assign i to j without any type conversion, the compiler will throw an error.

This brings us to an end of this tutorial. Please post your feedback and queries in the comments section.

**Next Tutorial - [Constants](https://golangbot.com/constants/)**