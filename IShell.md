shell
====

### Linux shell 截取字符变量的前 8 位,有方法如下:
```sh
1. expr substr “$a” 1 8
2. echo $a|awk ‘{print substr(,1,8)}’
3. echo $a|cut -c1-8
4. echo $
5. expr $a : ‘\(.\\).*’
6. echo $a|dd bs=1 count=8 2>/dev/null
```

### 按指定的字符串截取
#### 第一种方法:
```sh
${varible##*string} 从左向右截取最后一个 string 后的字符串
${varible#*string}从左向右截取第一个 string 后的字符串
${varible%%string*}从右向左截取最后一个 string 后的字符串
${varible%string*}从右向左截取第一个 string 后的字符串
“*”只是一个通配符可以不要
```
例子:
```sh
$ MYVAR=foodforthought.jpg
$ echo ${MYVAR##*fo}
rthought.jpg
$ echo ${MYVAR#*fo}
odforthought.jpg
```
#### 第二种方法:${varible:n1:n2}:截取变量 varible 从 n1 到 n2 之间的字符串。
可以根据特定字符偏移和长度,使用另一种形式的变量扩展,来选择特定子字符串。试着在 bash 中输入以下行:
```sh
$ EXCLAIM=cowabunga
$ echo ${EXCLAIM:0:3}
cow
$ echo ${EXCLAIM:3:7}
abunga
```
#### 第三种方法：按照指定要求分割:
比如获取后缀名
```sh
ls -al | cut -d “.” -f2
```

## function

shell 中的function的用法如下事例：

```sh
function fun_1(){
	command=$1
	arg=$2
	....
}
function fun_2(){
fun_1 command arg
}
```
## if
#### 字符串为空判断
```sh
if [ $(echo $command|wc -L) -eq 0 ];then
	show_usage
elif [ $(echo $arg|wc -L) -eq 0 ];then
	show_usage	
else
fi
```
#### 字符串相等
```sh
if [[ $command = "all" ]];then
else
fi
```

#### 文件判断
```sh
file=/home/xx/y.txt
if[! -f $file];then
else
	echo ""	
fi
```
#### 目录判断
```sh
file=/home/xx
if[! -d $file];then
else
	echo ""
fi
```

### until

```sh
until ((i > 100))
do
    ((sum += i))
    ((i++))
done
```

```sh
until mysql -h 127.0.0.1 -e "SELECT 1"; do sleep 1; done
```

### grep

未匹配到的时候会返回1,在有`set -ex`的时候会导致脚本退出

```sh
$ echo "anything" | grep e
### error
$ echo $?
1
$ echo "anything" | grep e || true
### no error
$ echo $?
0
### DopeGhoti's "no-op" version
### (Potentially avoids spawning a process, if `true` is not a builtin):
$ echo "anything" | grep e || :
### no error
$ echo $?
0
```



