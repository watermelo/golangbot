> * 原文地址：[Part 1: Introduction and Installation](https://golangbot.com/golang-tutorial-part-1-introduction-and-installation/)
> * 原文作者：[Naveen R](https://golangbot.com/about/)
> * 译者：咔叽咔叽
转载请注明出处。

This is the first tutorial in our [Golang tutorial series](https://golangbot.com/learn-golang-series/).

### What is Golang

Go also known as Golang is an open source, compiled and statically typed programming language created by google.

The primary focus of Golang is to make the development of highly available and scalable web apps simple and easy.

### Why Golang

Why would you choose Golang as your service side programming language when there are tonnes of other languages such as python, ruby, nodejs... that do the same job.

Here are some of the pros which I found in choosing Go

*   Concurrency is an inherent part of the language. As a result writing multithreaded programs is a piece of cake. This is achieved by Goroutines and channels which we will discuss later in the upcoming tutorials.
*   Golang is a compiled language. The source code is compiled to native binary. This is missing in interpreted languages such as JavaScript used in nodejs.
*   The language spec is pretty simple. The [entire spec](https://golang.org/ref/spec) fits in a page and you can even use it to write your own compiler :)
*   The go compiler supports static linking. All the go code can be statically linked into one big fat binary and it can be deployed in cloud servers easily without worrying about dependencies.

### Installation

Golang is supported on all the three platforms Mac, Windows and Linux. You can download the binary for the corresponding platform from [https://golang.org/dl/](https://golang.org/dl/)

#### Mac OS

Download the OS X installer from [https://golang.org/dl/](https://golang.org/dl/). Double tap to start the installation. Follow the prompts and this should have Golang installed in _/usr/local/go_ and would have also added the folder _/usr/local/go/bin_ to your PATH environment variable.

#### Windows

Download the MSI installer from [https://golang.org/dl/](https://golang.org/dl/). Double tap to start the installation and follow the prompts. This will install Golang in location _c:\\Go_ and will also add the directory _c:\\Go\\bin_ to your path environment variable.

#### Linux

Download the tar file from [https://golang.org/dl/](https://golang.org/dl/) and unzip it to /usr/local.

Add /usr/local/go/bin to the PATH environment variable. This should install go in linux.

In the next part [Golang tutorial part 2: Hello World](https://golangbot.com/hello-world/) of this series we will setup the go workspace and write our first Go program :)

Please provide your valuable feedback and comments. Thanks for reading :).

**Next tutorial - [Hello World](https://golangbot.com/hello-world/)**