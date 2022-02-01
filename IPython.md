python
====

> 目前python2.X已经不维护，各大操作系统已经陆续替换到python3

### 包安装

#### easy_install vs pip
easy_insall的作用和perl中的cpan, ruby中的gem类似，都提供了在线一键安装模块的傻瓜方便方式，而pip是easy_install的改进版, 提供更好的提示信息，删除package等功能。老版本的python中只有easy_install, 没有pip

#### pip install
```sh
$sudo apt install python3-pip
```
如果用sudo apt 安装python的库,会安装到/usr/lib/python/site-packages下.如果用pip3安装会安装到~/.local/lib/python3.7/site-packages
如果要修改pip的安装路径需要

```sh
export PYTHONPAT=$PYTHONPATH:/TOOLS/PYTHON_PATH/lib/python3.7/site-packages
export PYTHONUSERBASE=/TOOLS/PYTHON_PATH
```
### pip 源
add some code to `~/.config/pip/pip.conf`
```ini
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/
```
### python path

1. 建议python安装在指定的目录下，这个时候需要设置python的PYTHONPATH环境变量

2. PYTHONPATH的环境变量和--preifx的目录不一样
	如果--prefix=/TOOLS/pythonLibs
	则PYTHONPATH=/TOOLS/pythonLibs/lib/python2.7/site-packages
	否则无法安装成功，提示
	TEST FAILED: /TOOLS/pythonLibs/lib/python2.7/site-packages/ does NOT support .pth files
	error: bad install directory or PYTHONPATH
	
3. 配置好了环境变量后，采用如下方式安装

  ```sh
  python setup.py  install --prefix=/TOOLS/pythonLibs
  ```

  ```sh
  easy_install --prefix=/TOOLS/pythonLibs web.py
  ```

### setup

1. 基本安装
	进入到该文件的setup.py 目录下 ，打开cmd，并切换到该目录下
	
	```sh
	$python setup.py build 
	$python setup.py install
	```
	
	该安装会安装到 /usr/lib/python下,如果需要安装到当前用户下,需要执行如下命令
	
2. 安装到~/.local/lib/python下

   ```sh
     $python  setup.py install --user
   ```

   

此种安装方式会安装一个egg包进去

3. pip 安装

   ```sh
     $pip install -e 'pwd of setup.py'		
     $pip uninstall py-remote-deployer
   ```

     此种安装方式会安装一堆链接文件进去

### 私有成员
Python不像C++、Java、C#等有明确的公共、私有或受保护的关键字来定义成员函数或属性，它使用约定的单下划线“_"和"__"双下划线作为函数或属性的前缀来标识。使用单下划线还是双下划线，是有很大的区别的。

1. 单下划线的函数或属性，在类定义中可以调用和访问，类的实例可以直接访问，子类中可以访问；

2. 双下划线的函数或属性，在类定义中可以调用和访问，类的实例不可以直接访问，子类不可访问。


### pip
```sh
pip show tensorflow
pip list
```



## virtualenv

> virtualenv是一个与操作系统隔离的虚拟环境，可以让不同的软件使用不同的环境，防止版本覆盖。

 ```
sudo apt install virtualenv
 ```

 创建虚拟环境

 ```sh
virtualenv --no-site-packages --python=python3.6 venv
 ```

 `--no-site-packages`，这样，已经安装到系统Python环境中的所有第三方包都不会复制过来

```
virtualenv --python=python3.8 venv
```

