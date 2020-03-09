Go
====

### install

1. go的安装相对简单，下载源代码，按照说明安装即可。
2. 安装tools
   	A、在go1.2之后的版本，相关的工具移动到了tools这个子项目中
      	B、默认安装的go已经不包含gocode、godoc、oracle等工具，需要另行安装
      	C、安装golang tools首先需要下载golang的源代码，有如下两种途径
      		I、从code.google.com上下载
      		II、直接使用go get github.com/golang/tools
      		III、注意如上两种方式拿到的tools的包名是不一样的
      	D、这里有一个坑
      		I、在go install 或者 go build的时候只有main package【包含package main】才能生成可执行文件。
      		II、在go get github.com/golang/tools之后，在/TOOLS/software/go/gopath/src/github.com/golang/tools/oracle目录下运行 go build发现未生成任何可执行文件。
      		III、后来才发现，oracle的mian()在/TOOLS/software/go/gopath/src/github.com/golang/tools/cmd/oracle，故执行如下命令成功编译生成oracle
      			go install github.com/golang/tools/cmd/oracle，在生成的可执行文件在/TOOLS/software/go/gopath/bin/oracle
      	E、这里还有一个坑
      		I、在从github或者code.google.com拿下来的代码中，有import "golang.org/x/tools/go/ast/astutil"，但是其实根本没有这个包，不知道为什么
      		II、通过如下命令将这些依赖修改未对应的。
      			#：x=`find .|grep .go$`
      			#：for i in $x; do sed -i "s/golang.org\/x/github.com\/golang/g" $i; done

### install 1.14

1. 先安装默认的go，用于编译

   ```
   sudo apt install golang-go
   ```

2. 下载go远吗

   ```
   wget https://dl.google.com/go/go1.14.src.tar.gz
   ```

   

3. 编译 go

   ```
   tar -xzvf go1.14.src.tar.gz
   cd go1.14/src
   ./make.bash 
   ```

4. 编译 所有

   ```
   ./all.bash
   ```

5. GOPATH and GOROOT

   ```
   # the classpath of go
   export GOPATH=$HOME/GOPATH
   # the root og go sofware
   export GOROOT=$HOME/go1.14
   export PATH=$PATH:$GOROOT/bin
   ```

   