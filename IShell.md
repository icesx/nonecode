一、Linux shell 截取字符变量的前 8 位,有方法如下:
1.expr substr “$a” 1 8
2.echo $a|awk ‘{print substr(,1,8)}’
3.echo $a|cut -c1-8
4.echo $
5.expr $a : ‘\(.\\).*’
6.echo $a|dd bs=1 count=8 2>/dev/null
二
二、按指定的字符串截取
1、第一种方法:
${varible##*string} 从左向右截取最后一个 string 后的字符串
${varible#*string}从左向右截取第一个 string 后的字符串
${varible%%string*}从右向左截取最后一个 string 后的字符串
${varible%string*}从右向左截取第一个 string 后的字符串
“*”只是一个通配符可以不要
例子:
$ MYVAR=foodforthought.jpg
$ echo ${MYVAR##*fo}
rthought.jpg
$ echo ${MYVAR#*fo}
odforthought.jpg
2、第二种方法:${varible:n1:n2}:截取变量 varible 从 n1 到 n2 之间的字符串。
可以根据特定字符偏移和长度,使用另一种形式的变量扩展,来选择特定子字符串。试着在 bash 中输入以下行:
$ EXCLAIM=cowabunga
$ echo ${EXCLAIM:0:3}
cow
$ echo ${EXCLAIM:3:7}
abunga
三、按照指定要求分割:
比如获取后缀名
ls -al | cut -d “.” -f2
