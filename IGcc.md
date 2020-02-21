gcc
==================
## 命名

```
工具链有一个松散的名称约定arch[-vendor][-os]-abi：

arch适用于架构：arm，mips，x86，i686 ......
vendor是工具链供应商：苹果，
os适用于操作系统：linux，none（裸机）
abi适用于应用程序二进制接口约定：eabi，gnueabi，gnueabihf
```

## 编译安装
​	下载[gcc 下载](https://gcc.gnu.org/mirrors.html) 
​	suao apt install libmpc-dev zip
​	./configure --enable-checking=release --enable-languages=c,c++ --prefix=/usr/lib/gcc-6.3.0
​	make -j4
​	make install

## arm 交叉编译工具
https://releases.linaro.org/components/toolchain/binaries/

## 多版本切换
https://blog.csdn.net/ykr168age/article/details/61615212

### 安装

如果当前系统已经安装有gcc，一般安装在“/usr/lib/gcc”下，如果需要多版本，需要如下操作
40、50为优先级	

```
whereis gcc
/usr/bin/gcc
//gcc 安装目录，一般也是一个链接文件，链接到/usr/bin/gcc-5

sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.9 30
sudo update-alternatives --install /usr/bin/gcc gcc /usr/lib/gcc-6.3.0/bin/gcc 31
sudo update-alternatives --config gcc
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-5 30 
sudo update-alternatives --install /usr/bin/g++ g++ /usr/lib/gcc-6.3.0/bin/g++ 31
sudo update-alternatives --config g++
gcc -v

sudo update-alternatives --remove gcc /usr/bin/gcc-4.9

```
 至此 gcc版本切换完成
 下一步需要将libstdc++.so.6 链接完成。

```
cd /usr/lib/aarch64-linux-gnu/
sudo ln -s /usr/lib/gcc-6.3.0/lib64/libstdc++.so.6.0.22 libstdc++.so.6.0.22
sudo ln -s libstdc++.so.6.0.22 libstdc++.so.6

```
 ln -s 

### 配置libstdc++.so.6
存放在
/usr/lib/aarch64-linux-gnu/libstdc++.so.6
/usr/lib/x86_64-linux-gnu/libstdc++.so.6
## ldconfig
ldconfig命令的用途主要是在默认搜寻目录/lib和/usr/lib以及动态库配置文件/etc/ld.so.conf内所列的目录下，搜索出可共享的动态链接库（格式如lib*.so*）,进而创建出动态装入程序(ld.so)所需的连接和缓存文件

1. 往/lib和/usr/lib里面加东西，是不用修改/etc/ld.so.conf的，但是完了之后要调一下ldconfig，不然这个library会找不到。
2. 想往上面两个目录以外加东西的时候，一定要修改/etc/ld.so.conf，然后再调用ldconfig，不然也会找不到。
	*比如安装了一个mysql到/usr/local/mysql，mysql有一大堆library在/usr/local/mysql/lib下面，这时就需要在/etc/ld.so.conf下面加一行/usr/local/mysql/lib，保存过后ldconfig一下，新的library才能在程序运行时被找到。*
3. 如果想在这两个目录以外放lib，但是又不想在/etc/ld.so.conf中加东西（或者是没有权限加东西）。那也可以，就是export一个全局变量LD_LIBRARY_PATH，然后运行程序的时候就会去这个目录中找library。一般来讲这只是一种临时的解决方案，在没有权限或临时需要的时候使用。
4. ldconfig做的这些东西都与运行程序时有关，跟编译时一点关系都没有。编译的时候还是该加-L就得加，不要混淆了。
5. 总之，就是不管做了什么关于library的变动后，最好都ldconfig一下，不然会出现一些意想不到的结果。不会花太多的时间，但是会省很多的事。
6. 再有，诸如libdb-4.3.so文件头中是会含有库名相关的信息的（即含“libdb-4.3.so”，可用strings命令察看），因此仅通过修改文件名以冒充某已被识别的库（如libdb-4.8.so）是行不通的。为此可在编译库的Makefile中直接修改配置信息，指定特别的库名。

## ld.so.cache
ld.so.cache 保持了当前系统的库加载路径，类似classpath

	strings /etc/so.cache

可以查看库的加载路径。
## glibc
linux 下的glibc
	

	./lib/arm-linux-gnueabihf/libc-2.23.so
