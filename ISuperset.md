Superset
======

[官网](https://superset.apache.org/docs/intro)

## 安装

**Debian and Ubuntu**

The following command will ensure that the required dependencies are installed:

```
sudo apt-get install build-essential libssl-dev libffi-dev python-dev python-pip libsasl2-dev libldap2-dev
```

In Ubuntu 20.04 the following command will ensure that the required dependencies are installed:

```sh
sudo apt-get install build-essential libssl-dev libffi-dev python3-dev python3-pip libsasl2-dev libldap2-dev
```

### Python Virtual Environment

```
sudo apt install virtualenv
virtualenv --no-site-packages --python=python3.8 venv
```



### Installing and Initializing Superset

```
pip3 install setuptools-rust
```

First, start by installing `apache-superset`:

```
pip install apache-superset
```

```
pip install pillow
```

Then, you need to initialize the database:

    export LC_ALL=C.UTF-8
    export LANG=C.UTF-8

Finish installing by running through the following commands:

```
superset db upgrade
```



```
superset fab create-admin
superset load_examples
superset init
```

test

```
superset run -p 8088 -h bjrdc49 --with-threads --reload --debugger
```

If everything worked, you should be able to navigate to `hostname:port` in your browser (e.g. locally by default at `localhost:8088`) and login using the username and password you created.

### start

superset默认命令启动是开发模式，生产模式需要使用gunicron



```sh
sudo apt install gunicorn
```



```shell
gunicorn  -w 10 -k gevent --timeout 120 -b  0.0.0.0:8088  "superset.app:create_app()" --daemon
```



### error: can't find Rust compiler

```sh
pip install --upgrade pip 
```



### Python.h: No such file or directory

```sh
sudo apt install python3-dev
```



### "Flask-Caching: CACHE_TYPE is set to null, "

## Kylin



```
kylin://<username>:<password>@<hostname>:<port>/<project>?<param1>=<value1>&<param2>=<value2>
Apache Kylin
```

```
kylin://ADMIN:KYLIN@bjrdc42:7070/learn_kylin
```

## 图表共享

