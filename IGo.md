Go
====

## install

1. go的安装相对简单，下载源代码，按照说明安装即可。

2. 安装tools

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



## 编译

1. go build 产生的可执行文件是自带环境的，不需要动态链接（cp到其他机器上可以直接执行）。
2. 自动获取依赖包

```
#当前目录下执行：
go get ./...
```



