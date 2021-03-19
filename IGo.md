Go
====

## install

### go

go的安装相对简单，下载源代码，按照说明安装即可。

### 安装tools

1. 在go1.2之后的版本，相关的工具移动到了tools这个子项目中

2. 默认安装的go已经不包含gocode、godoc、oracle等工具，需要另行安装

3. 安装golang tools首先需要下载golang的源代码，有如下两种途径

    1. 从code.google.com上下载

    2. 直接使用go get github.com/golang/tools  		
    3. 注意如上两种方式拿到的tools的包名是不一样的

4. 这里有一个坑
      		

    1. 在go install 或者 go build的时候只有main package【包含package main】才能生成可执行文件。
          		

    2. 在go get github.com/golang/tools之后，在/TOOLS/software/go/gopath/src/github.com/golang/tools/oracle目录下运行 go build发现未生成任何可执行文件。  		

    3. 后来才发现，oracle的mian()在/TOOLS/software/go/gopath/src/github.com/golang/tools/cmd/oracle，故执行如下命令成功编译生成oracle

        ```
        go install github.com/golang/tools/cmd/oracle
        ```

        在生成的可执行文件在/TOOLS/software/go/gopath/bin/oracle

5. 这里还有一个坑
      		

    1. 在从github或者code.google.com拿下来的代码中，有import "golang.org/x/tools/go/ast/astutil"，但是其实根本没有这个包，不知道为什么
          		

    2. 通过如下命令将这些依赖修改未对应的。

        ```sh
        x=`find .|grep .go$`
        for i in $x; do sed -i "s/golang.org\/x/github.com\/golang/g" $i; done
        ```

        
        
### gocode

> 提供代码提醒的，如果是vscode的话可以不用

## install 1.14

1. 先安装默认的go，用于编译

   ```sh
   sudo apt install golang-go
   ```

2. 下载go远吗

   ```sh
   wget https://dl.google.com/go/go1.14.src.tar.gz
   ```

   

3. 编译 go

   ```sh
   tar -xzvf go1.14.src.tar.gz
   cd go1.14/src
   ./make.bash 
   ```

4. 编译 所有

   ```sh
   ./all.bash
   ```

5. GOPATH and GOROOT

   ```sh
   # the classpath of go
   export GOPATH=$HOME/GOPATH
   # the root og go sofware
   export GOROOT=$HOME/go1.14
   export PATH=$PATH:$GOROOT/bin
   ```


### GOPATH

1. 工作区用于放置 Go 语言的**源码文件（source file）**以及安装（install）后的**归档文件（archive file，也就是以“.a”为扩展名的文件）**和**可执行文件（executable file）**
2. 在工作区中，一个代码包的导入路径实际上就是从 src 子目录，到该包的实际存储位置的相对路径
3. 源码文件通常会被放在某个工作区的 src 子目录下。
4. 那么在安装后如果产生了归档文件，就会放进该工作区的 pkg 子目录；如果产生了可执行文件，就可能会放进该工作区的 bin 子目录。

## code

[参考](https://wizardforcel.gitbooks.io/go42/content/)

### helloworld

```go
package main
import "fmt"
func main() {
    fmt.Println("hello world")
}
```

```sh
go run hw.go
```

### Go程序的编译

如果想要构建一个程序，则包和包内的文件都必须以正确的顺序进行编译。包的依赖关系决定了其构建顺序。

属于同一个包的源文件必须全部被一起编译，一个包即是编译时的一个单元，因此根据惯例，每个目录都只包含一个包。

如果对一个包进行更改或重新编译，所有引用了这个包的客户端程序都必须全部重新编译。 Go 中的包模型采用了显式依赖关系的机制来达到快速编译的目的，编译器会从后缀名为 .go 的对象文件（需要且只需要这个文件）中提取传递依赖类型的信息。

如果 A.go 依赖 B.go，而 B.go 又依赖 C.go：

> 编译 C.go，B.go，然后是 A.go。
> 为了编译 A.go，编译器读取的是 B.go 而不是 C.go。

这种机制对于编译大型的项目时可以显著地提升编译速度，每一段代码只会被编译一次。

在Go语言中，和编译有关的命令主要是go run ,go build , go install这三个命令。

go run只能作用于main包文件，先运行compile 命令编译生成.a文件，然后 link 生成最终可执行文件并运行程序，这过程的产生的是临时文件，在go run 退出前会删除这些临时文件（含.a文件和可执行文件）。最后直接在命令行输出程序执行结果。go run 命令在第二次执行的时候，如果发现导入的代码包没有发生变化，那么 go run 不会再次编译这个导入的代码包，直接进行链接生成最终可执行文件并运行程序。

go install用于编译并安装指定的代码包及它们的依赖包，并且将编译后生成的可执行文件放到 bin 目录下（$GOPATH/bin），编译后的包文件放到当前工作区的 pkg 的平台相关目录下。

go build用于编译指定的代码包以及它们的依赖包。如果用来编译非main包的源码，则只做检查性的编译，而不会输出任何结果文件。如果是一个可执行程序的源码（即是 main 包），这个过程与go run 大体相同，除了会在当前目录生成一个可执行文件外。

使用go build时有一个地方需要注意，对外发布编译文件如果不希望被人看到源代码，请使用go build -ldflags 命令，设置编译参数-ldflags "-w -s" 再编译后发布。避免使用gdb来调试而清楚看到源代码。

### 接口

在Go语言中，如果接口的所有方法在某个类型方法集中被实现，则认为该类型实现了这个接口。

类型不用显式声明实现了接口，只需要实现接口所有方法，这样的隐式实现解藕了实现接口的包和定义接口的包。



## 编译

1. go build 产生的可执行文件是自带环境的，不需要动态链接（cp到其他机器上可以直接执行）。
2. 自动获取依赖包

```
#当前目录下执行：
go get ./...
```



## vscode

### modules

go 在1.11版本后增加了modules，用于支持脱离GOPATH的开发模式（GOPATH下项目的代码必须放到GOPATH下）。

### vscode配置

1. 安装插件[The VS Code Go extension](https://marketplace.visualstudio.com/items?itemName=golang.go)

2. 配置

   在settings.json中增加如下配置

   ```json
       "go.toolsGopath": "/TOOLS/GOPATH",
       "go.goroot": "/TOOLS/software/go",
       "gopls": {
               "experimentalWorkspaceModule": true,
       },
   ```

   `experimentalWorkspaceModule`一个workspace支持多个go.mod文件

3. 如果使用modules的开发模式，则不可以在操作系统中配置GOPATH环境变量

### 项目目录结果

src下每个目录都是一个module。

```
└── src
    ├── example
    │   └── langage.go
    ├── gobase
    │   ├── add.go
    │   ├── defer.go
    │   ├── gobase_test.go
    │   ├── go.mod
    │   ├── interface.go
    │   ├── list.go
    │   ├── math.go
    │   ├── print.go
    │   └── swap.go
    ├── gor
    │   ├── gor.go
    │   └── gor_test.go
    ├── main
    │   ├── go.mod
    │   └── main.go
    ├── mysql
    │   ├── mysql.go
    │   └── mysql_test.go
    └── server
        ├── go.mod
        └── http8080.go
```

### module操作

详细参考 [create module](https://golang.org/doc/tutorial/create-module#start)  [call-mdule-code](https://golang.org/doc/tutorial/call-module-code)

