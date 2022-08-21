# Superset

[官网](https://superset.apache.org/docs/intro)

## 安装

### **Debian and Ubuntu**

The following command will ensure that the required dependencies are installed:

```
sudo apt-get install build-essential libssl-dev libffi-dev python-dev python-pip libsasl2-dev libldap2-dev
```

In Ubuntu 20.04 the following command will ensure that the required dependencies are installed:

```sh
sudo apt-get install build-essential libssl-dev libffi-dev python3-dev python3-pip libsasl2-dev libldap2-dev
```

#### Python Virtual Environment

```shell
sudo apt install virtualenv
virtualenv --python=python3.8 venv
```



#### Installing and Initializing Superset

```sh
source venv/bin/activate
```

```
pip3 install setuptools-rust
```

First, start by installing `apache-superset`:

如果使用mysql作为数据库，需要先安装mysql的库

```
sudo apt install libmysqlclient-dev
pip3 install mysqlclient
```



```
pip3 install apache-superset
```

```
pip3 install pillow
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

#### test

```
superset run -p 8088 -h bjrdc49 --with-threads --reload --debugger
```

If everything worked, you should be able to navigate to `hostname:port` in your browser (e.g. locally by default at `localhost:8088`) and login using the username and password you created.

#### start

superset默认命令启动是开发模式，生产模式需要使用gunicron



```sh
sudo apt install gunicorn
```



```shell
gunicorn  -w 10 -k gevent --timeout 120 -b  0.0.0.0:8088  "superset.app:create_app()" --daemon
```

### 错误处理

#### error: can't find Rust compiler

```sh
pip install --upgrade pip 
```



#### Python.h: No such file or directory

```sh
sudo apt install python3-dev
```



#### Flask-Caching: CACHE_TYPE is set to null,



#### ERROR: flask-appbuilder 3.4.0 has requirement Flask-WTF<0.15.0,>=0.14.2, but you'll have flask-wtf 1.0.0 which is incompatible.



#### UnicodeDecodeError: 'utf-8' codec can't decode byte 0xd2 in position 0: invalid continuation byte

  File "/cloud/venv/lib/python3.8/site-packages/SQLAlchemy_Utils-0.37.9-py3.8.egg/sqlalchemy_utils/types/encrypted/encrypted_type.py", line 128, in decrypt
    decrypted = decrypted.decode('utf-8')
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xd2 in position 0: invalid continuation byte

当superset切换数据库的时候，如果数据库是从老版本dump过来的话，可能会在`superset init`的时候出现这个问题，[解决办法](https://github.com/apache/superset/issues/8538#issuecomment-1046238602)是

1. Change SECRET_KEY on superset_config.py
2. Access psql client and connect to the superset DB (\connect db_name;)
3. `update dbs set password = null, encrypted_extra=null;`
4. superset db upgrade
5. superset init
6. superset run
7. Update password on Superset: Data > Databases (I'm using 1.4.1 btw)

## 数据源

### Kylin

```
kylin://<username>:<password>@<hostname>:<port>/<project>?<param1>=<value1>&<param2>=<value2>
Apache Kylin
```

```
kylin://ADMIN:KYLIN@bjrdc42:7070/learn_kylin
```

### mysql



## 配置

在PYTONPATH `venv/lib/python3.8/site-package/`下放置`superset_config.py`文件

### 语言设置

修改`superset_config.py`文件，设置`BABEL_DEFAULT_LOCALE = "zh"`

```python
BABEL_DEFAULT_LOCALE = "zh"
# Your application default translation path
#BABEL_DEFAULT_FOLDER = "superset/translations"
BABEL_DEFAULT_FOLDER = "superset/translations"
# The allowed translation for you app
LANGUAGES = {
    "en": {"flag": "us", "name": "English"},
   # "es": {"flag": "es", "name": "Spanish"},
   # "it": {"flag": "it", "name": "Italian"},
   # "fr": {"flag": "fr", "name": "French"},
    "zh": {"flag": "cn", "name": "Chinese"},
   # "ja": {"flag": "jp", "name": "Japanese"},
   # "de": {"flag": "de", "name": "German"},
   # "pt": {"flag": "pt", "name": "Portuguese"},
   # "pt_BR": {"flag": "br", "name": "Brazilian Portuguese"},
   # "ru": {"flag": "ru", "name": "Russian"},
   # "ko": {"flag": "kr", "name": "Korean"},
   # "sl": {"flag": "si", "name": "Slovenian"},
}
# Turning off i18n by default as translation in most languages are
# incomplete and not well maintained.
#LANGUAGES = {}
```

### 存储数据库

```python
SQLALCHEMY_DATABASE_URI = 'mysql://superset:superset123@bjrdc60/hav_superset'
```



### thumbnail

#### 原理

1. 使用redis存储缩略图
2. 使用webdriver生成缩略图，注意chromedriver需要和chrome版本对应
3. 异步通知celery创建缩略图

#### webdriver安装

```sh
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb && \
    apt install -y ./google-chrome-stable_current_amd64.deb
```

chrome安装成功后，查看chrome的版本号，如版本号未96.xx.x.x

则在`https://chromedriver.storage.googleapis.com/`的xml中查看对应的99版本的chromedriver的版本，如为

```xml
<Contents>
<Key>96.0.4664.18/chromedriver_linux64.zip</Key>
<Generation>1635142532134035</Generation>
<MetaGeneration>1</MetaGeneration>
<LastModified>2021-10-25T06:15:32.167Z</LastModified>
<ETag>"e8b409daf617e69cbf9a325606d65375"</ETag>
<Size>9981693</Size>
</Contents>
```



```sh
    wget https://chromedriver.storage.googleapis.com/96.0.4664.18/chromedriver_linux64.zip && \
    unzip chromedriver_linux64.zip && \
    chmod +x chromedriver && \
    mv chromedriver /usr/bin && \
    rm -f google-chrome-stable_current_amd64.deb chromedriver_linux64.zip
```

#### 配置

superset_config.py，中增加如下配置

```python
FEATURE_FLAGS = {
    "THUMBNAILS": True,
}
from superset.typing import CacheConfig
THUMBNAIL_CACHE_CONFIG: CacheConfig = {
    'CACHE_TYPE': 'redis',
    'CACHE_DEFAULT_TIMEOUT': 24 * 60 * 60,
    'CACHE_KEY_PREFIX': 'thumbnail_',
    'CACHE_NO_NULL_WARNING': True,
    'CACHE_REDIS_URL': 'redis://localhost:6379/0'
}

class CeleryConfig(object):
    BROKER_URL = "redis://localhost:6379/1"
    CELERY_IMPORTS = ("superset.sql_lab", "superset.tasks", "superset.tasks.thumbnails",)
    CELERY_RESULT_BACKEND = "redis://localhost:6379/1"
    CELERYD_PREFETCH_MULTIPLIER = 10
    CELERY_ACKS_LATE = True

CELERY_CONFIG = CeleryConfig

DATA_CACHE_CONFIG = {
    'CACHE_TYPE': 'redis',
    'CACHE_DEFAULT_TIMEOUT': 60 * 60 * 24, # 1 day default (in secs)
    'CACHE_KEY_PREFIX': 'superset_results',
    'CACHE_REDIS_URL': 'redis://localhost:6379/2',
}
#此处要写外网地址，不知道原因
WEBDRIVER_BASEURL = "http://bjrdc61:8088/"
WEBDRIVER_TYPE= "chrome"
WEBDRIVER_OPTION_ARGS = [
        "--force-device-scale-factor=2.0",
        "--high-dpi-support=2.0",
        "--headless",
        "--disable-gpu",
        "--disable-dev-shm-usage",
        "--no-sandbox",
        "--disable-setuid-sandbox",
        "--disable-extensions",
        "--disable-dev-shm-usage",
        "--disable-extensions",
        "--disable-dev-shm-usage",
        ]
```

#### 启动celery

```
celery --app=superset.tasks.celery_app:app worker --pool=prefork -O fair -c 3&
```

#### 替代方案

由于superset的缩略图使用celery，而找了先关的资料，并没有找到将celery分布式部署的方案，而且此种截图的方案并不稳定。

可以采用`selenium`使用pyhton自己构建截图的方案



## 样例

superset 自带样例，可以使用如下命令下载

```
superset load_examples
```

但是由于需要去github上下载，经常会timeout。可以使用如下方法解决

1. 从github上面下载文件集 examples-data-master.zip https://github.com/apache-superset/examples-data-maser

2. 将下载的压缩文件解压后放到nginx下，如localhost:8888/examples-data-master。注：这个地址要列出所有的压缩包内容

3. 修改superset的helper.py并修改ia

   ```
   vi /superset/examples/helpers.py
   # BASE_URL = "https://github.com/apache-superset/examples-data/blob/master/"
   BASE_URL="http://localhost:8888/examples-data-master/"
   ```

4. 重新执行`superset load_examples`

## 开发

### 几个重要概念

#### superset

#### superset-core

#### superset-ui

#### superset-frontend



### 获取代码

```shell
git clone https://github.com/apache/superset.git
git switch origin/1.3
```

### 官方编译方式

官方编译编译superset和superset-frontend两个步骤。

#### 编译superset

1. mysql

   ```
   sudo apt-get install libmysqlclient-dev
   ```

   

2. 穿件虚拟环境

   ```sh
   virtualenv --python=python3.8 /TOOLS/py_venv/venv_superset
   ```

3. 编译superset

   ```sh
   source /TOOLS/py_venv/venv_superset/bin/activate
   ```

   ```sh
   
   # Install external dependencies
   pip install -r requirements/testing.txt
   
   # Install Superset in editable (development) mode
   pip install -e .
   
   # Initialize the database
   superset db upgrade
   
   # Create an admin user in your metadata database (use `admin` as username to be able to load the examples)
   superset fab create-admin
   
   # Create default roles and permissions
   superset init
   
   # Load some data to play with.
   # Note: you MUST have previously created an admin user with the username `admin` for this command to work.
   superset load-examples
   ```

4. 启动

   ```sh
   superset run -p 8088 --with-threads --reload --debugger
   ```

   至此superset启动了，但是由于frontend未编译，故在`superset/static/assets/`下是空的，需要将frongtend编译，编译后会自动将编译后的资源复制到assets下。

#### 编译superset-frontend

```sh
cd superset/superset-frontend
cd superset-frontend
nvm install --lts
nvm use --lts
# Install dependencies from `package-lock.json`
npm ci
#local development assets, with sourcemaps and hot refresh support
npm run build
```

编译完成后，静态资源会自动的复制到`superset/static/assets`目录下

#### 开发superset-frontend

1. 启动frontend

   The dev server by default starts at `http://localhost:9000` and proxies the backend requests to `http://localhost:8088`. It's possible to change these settings:

   ```sh
   # Start the dev server at http://localhost:9000
   npm run dev-server
   ```

   或者

   ```sh
   # Run the dev server on a non-default port
   npm run dev-server -- --devserverPort=9001
   
   # Proxy backend requests to a Flask server running on a non-default port
   npm run dev-server -- --supersetPort=8081
   
   # Proxy to a remote backend but serve local assets
   npm run dev-server -- --superset=https://superset-dev.example.com
   ```

3. Other npm commands

   Alternatively, there are other NPM commands you may find useful:

   1. `npm run build-dev`: build assets in development mode.
   2. `npm run dev`: built dev assets in watch mode, will automatically rebuild when a file changes



### 启动方式

[参考](https://apache-superset.readthedocs.io/en/latest/installation.html)

#### 启动superset（pycharm）

superset/cli.py 使用click库作为命令行工具.当执行`superset run..`命令的时候，其实是跑到了`flask/cli.py`里执行了flask的命令。

1. 安装pycharm

2. File->open-指定到superset的clone地址

3. 打开到superset/tests目录下编写启动superset的代码，superset_run_test.py

   ```python
   from superset.cli import superset
   
   if __name__ == '__main__':
       superset(['run', '-p', '8088', '--debugger'])
   
   ```

4. 执行该代码即可启动superset

5. superset-frontend需要另行启动

#### 启动superset-frontend（vscode）

vscode中

f1->npm:run script->dev-server

#### 

### 前端原理

superset的前端分为superset-frontend 和superset/superset。两个项目没有完全解耦。其中

1. superset/superset是一个flask的项目，使用flask的模板技术进行前端的页面的管理。
2. superset-frontend是react项目，纯静态的项目。

#### superset flask





#### superset-frontend

使用react开发的前端应用，经测试是不能直接使用的，编译后，需要cp（自动）到`superset/superset/static/assets/`目录下。

使用webpack进行开发模式运行的时候，其实是启动了一个webpack的proxy，该代理会将用户请求代理到**superset**后端也就是python开发的flask应用（原理尚未完全搞明白）。



#### flask代理

superset-frontend通过webpack的proxy来对flask的html进行替换，替换为webpack提供的最新的开发模式的js和css

### superset-ui

1. @superset-ui/core/lib/connection

   实现与superset后端restful通信的库
   
2. @superset-ui**的源代码在哪里呢**？

### 修改urlbase

#### 基本原理

superset的地址是前端和后端紧密配合的，前端虽然是react开发的应用，但是却不是真正的前后端分离。基本的配合方式是

1. superset通过FAB进行整个后端页面的构建，因此菜单、权限、链接等均通过FAB实现。
2. 菜单创建后，菜单中的页面的内容是通过falsk的模板将react的资源（js、css）等导入到页面。
3. 当页面渲染的时候，react首先后去检查当前页面的url和react中的路由对应，就会进行页面的绘制。
4. 需要注意的是，有一些页面的菜单是react绘制的。

superset-flask的后端应用（FAB）会通过模板技术发布相关的web地址，但是只在框架层面，主要是菜单。

菜单中的地址是可以通过修改python代码来实现修改。

#### 后端修改

故增加前缀的，有如下修改修改

1. config.py 添加

   ```python
   URL_PREFIX = "/superset"
   ```

2. create dataviews_utils.py

   ```python
   # coding:utf-8
   # Copyright (C)
   # Author: I
   # Contact: 12157724@qq.com
   import logging
   
   from superset import app
   
   default_url_prefix = app.config['URL_PREFIX']
   logger = logging.getLogger(__name__)
   
   
   def base_url(route_base, view):
       if route_base is None:
           route_base = "/" + view.__class__.__name__.lower()
           logger.debug(
               "route_base is none so set to its class name's lower " + route_base)
       if route_base.startswith(default_url_prefix) is False:
           route_base = default_url_prefix + route_base
           logger.info("route_base is not startwith 'superset' so add it: " +
                       route_base)
       else:
           logger.info("do nothing for:" + route_base)
       return route_base
   
   ```

   

3. superset/core.py

   修改所有的`redirect`增加前缀

   ```python
   return redirect(default_url_prefix+"/")
   ```

4. superset/views/base.py

   ```python
   class SupersetModelView(ModelView):
       page_size = 100
       list_widget = SupersetListWidget
   
       # ICESX add this method,to add /superset to all modelview
       def __init__(self, **kwargs):
           super().__init__(**kwargs)
           self.route_base = base_url(self.route_base, self)
   
   
   ```

   ```python
   class BaseSupersetView(BaseView):
       # ICESX add this for add superset for all view
       def __init__(self, **kwargs):
           super().__init__(**kwargs)
           base_url(self.route_base, self)
   
   ```

5. superst/initialization/\_\_init\_\_.py 中修改所有的`add_link`(add_link是FAB设置菜单和链接的函数)，类似如下代码

   ```python
           appbuilder.add_link(
               "Home",
               label=__("Home"),
               href=default_url_prefix+"/welcome/",
               cond=lambda: bool(appbuilder.app.config["LOGO_TARGET_PATH"]),
           )
   ```

   

6. superset/views/sql_lab.py

   ```python
   class SqlLab(BaseSupersetView):
       """The base views for Superset!"""
   
       @expose("/my_queries/")
       @has_access
       def my_queries(self) -> FlaskResponse:  # pylint: disable=no-self-use
           """Assigns a list of found users to the given role."""
           # return redirect("/savedqueryview/list/?_flt_0_user={}".format(g.user.get_id())) //ICESX
           return redirect("/superset/savedqueryview/list/?_flt_0_user={}".format(g.user.get_id()))
   
   ```

7. superset/views/database|dashboard|/views.py

   `return redirect(`

   replace all to 

   `return redirect(default_url_prefix+`

#### 前端修改

当浏览器的地址发生变化的时候，会触发react的渲染，也就是说后端expose的地址其实必须和前端需要渲染的路由匹配的。而susperset-frontend中有2个地方和路由相关的



1. add file dataviews/config.ts

   ```typescript
   const baseConfig = {
       "base_url_prefix": "/superset"
   }
   export default baseConfig;
   
   ```

   

2. src/views/routes.tsx

   ![image-20220317082456606](/ICESX/ISunflower/nonecode/ISuperset.assets/image-20220317082456606.png)

3. /src/views/CRUD/data/common.ts

   ![image-20220317081903739](/ICESX/ISunflower/nonecode/ISuperset.assets/image-20220317081903739.png)

4. /src/views/CRUD/chart/ChartList.tsx

   ```typescript
         onClick: () => {
           window.location.assign('/chart/add');
         },
   ```

   ```typescript
         onClick: () => {
           window.location.assign(baseConfig.base_url_prefix+'/chart/add');
         },
   ```

5. src/views/CRUD/dashboard/DashboardList.tsx

   ![image-20220317082147706](/ICESX/ISunflower/nonecode/ISuperset.assets/image-20220317082147706.png)

   

6. src/views/CRUD/data/query/QueryList.tsx

   ![image-20220317082237339](/ICESX/ISunflower/nonecode/ISuperset.assets/image-20220317082237339.png)

7. src/views/CRUD/welcome/ChartTable.tsx

   ![image-20220317082332380](/ICESX/ISunflower/nonecode/ISuperset.assets/image-20220317082332380.png)

   

8. src/views/CRUD/welcome/DashboardTable.ts

   ![image-20220317082414651](/ICESX/ISunflower/nonecode/ISuperset.assets/image-20220317082414651.png)

   



## 自定义插件

### install yarn

```
npm install --global yarn
yarn config set cache-folder /TOOLS/yarn/cache
```



### Install Superset-UI

```
git clone https://github.com/apache-superset/superset-ui
cd superset-ui
yarn install
```



### Build Your "Hello, World"

### *generate* some code!

```
cd superset-ui
mkdir plugin-chart-hello-world
cd plugin-chart-hello-world
yo @superset-ui/superset
```

![img](https://superset.apache.org/images/plugin-1-yeoman-select.png)

1. Give it a name (in our case, go with the default, based on the folder name):

   

2. Give it a description (again, default is fine!)

   

3. Choose which type of React component you want to make (Class, or Function component).

   

4. Select whether you'd like your visualization to be timeseries-based or not

   

5. Select whether or not you want to include badges at the top of your README file (really only needed if you intend to contribute your plugin to the `superset-ui` repo).

   

6. Admire all the files the generator has created for you. Note that EACH of these is chock full of comments about what they're for, and how best to use them.

   

### Add your Plugin to Superset (with NPM Link)

Now, we want to see this thing actually RUN! To do that, we'll add your package to Superset and embrace the magic power of `npm link` to see it in-situ, without needing to **build** the plugin, or open any PRs on Github.

1. Add your package to the `package.json` file in `superset/superset-frontend`.

   ![img](ISuperset.assets/plugin-8-package-json.png)

2. Add your plugin to the `MainPreset.js` file (located in `superset/superset-frontend/src/visualizations/presets/MainPreset.js`) in two places, alongside the other plugins.

![img](https://superset.apache.org/images/plugin-9-mainpreset-import.png)

![img](ISuperset.assets/plugin-9-mainpreset-register.png)

3. `npm run dev-server`. You'll know it worked if you see a line stating `[Superset Plugin] Use symlink source for @superset-ui/plugin-chart-hello-world @ ^0.0.0`.

**NOTE:** Dynamic import is a work in progress. We hope you won't even need to DO this soon. We'll be blogging again when that day comes, we assure you. In short, we have a goal to make editing `package.json` and `MainPreset.js` unnecessary, so all the code changes are made in ONE repo.

### See it with your own eyes!

You should now be able to go to the Explore view in your local Superset and add a new chart! You'll see your new plugin when you go to select your viz type.

![img](ISuperset.assets/plugin-10-hello-thumbnail.png)

Now you can load up some data, and you'll see it appear in the plugin!

![img](https://superset.apache.org/images/plugin-11-explore-view.png)

The plugin also outputs three things to your browser's console:

- `formData`, a.k.a. everything sent into your viz from the controls
- `props`, as output from the `transformProps` file for your plugin's consumption
- The actual HTML element, which your plugin has hooks into for any necessary DOM maniupluation

### build

如果不build的化，插件复制到superset-ui中，虽然在开发模式下可以运行，但是无法将superset-frontend，build到superset中，需要在插件复制到superset-ui之前`npm run build`

github上我有提一个[commit](https://github.com/apache/superset/issues/10433)



## echars插件修改

### 插件目录结构说明

```
├── package.json
├── README.md
├── src
│   ├── HelloWorld.tsx
│   ├── images
│   │   └── thumbnail.png
│   ├── index.ts
│   ├── plugin
│   │   ├── buildQuery.ts
│   │   ├── controlPanel.ts
│   │   ├── index.ts
│   │   └── transformProps.ts
│   └── types.ts
├── test
│   ├── index.test.ts
│   └── plugin
│       ├── buildQuery.test.ts
│       └── transformProps.test.ts
├── tsconfig.json
└── types
    └── external.d.ts
```

1. images 插件缩略图
2. index.js 插件模块的入口，export 插件，组装如下三个重要文件。
3. plugin/buildQuery.ts 插件用于设置查询条件的页面
4. plugin/controlPanel.ts 插件自定义的页面
5. **plugin/transformProps.ts**插件绘图控制的部分，将数据和绘图参数传递给绘图模块的过程。

### echats-pie插件解读

#### 目录结构

superset/superset-frontend/node_modules/@superset-ui/plugin-chart-echarts/esm/Pie

```
├── buildQuery.js
├── controlPanel.js
├── EchartsPie.js
├── images
│   ├── Pie1.jpg
│   ├── Pie2.jpg
│   ├── Pie3.jpg
│   ├── Pie4.jpg
│   └── thumbnail.png
├── index.js
├── transformProps.js
└── types.js
```

#### 变量绑定

在插件中采用sank的命名规范，在superset会将其自动转为chamel的命名方式。



## 图表共享



## 数据迁移

### 数据库配置

superset默认的数据库是在sqlite中。配置文件在superset/config.py文件文件中。

```python
SQLALCHEMY_DATABASE_URI = "sqlite:///" + os.path.join(DATA_DIR, "superset.db")
```

如果需要将数据库迁移到mysql，需要在PYTHONPATH下创建`superset_conf.py`文件，其中包含内容

```python
SQLALCHEMY_DATABASE_URI = 'mysql://superset:xxxxx@bjrdc60/hav_superset'
```

### 数据迁移

1. sqlite3mysql

2. 数据迁移过程首先使用工具`sqlite3mysql`迁移数据库到mysql，但是发现有报错，无法执行完成。

   ```sh
   sqlite3mysql -f superset-11-13.db.dump.db -d hav_superset -u bjrdc -h bjrdc60 --mysql-password xxx
   ```

3. 人工

   后经人工修改sqlite导出的sql文件，然后使用mysql source进行导入，最终修改了所有的错误后，导入成功，但是superset无法启动。

4. supert导出再导入

   ```sh
   superset export-datasources >datasources.yaml
   superset import-datasources -p datasources.yaml
   superset export-dashboards >dashboards.json
   superset import-dashboards -p dashboards.json
   ```

   注意：导出的文件中，可能有垃圾行，需要删除

   导入后，除用户信息丢失外，其他的内容都在。

## 部署

### nginx+superset

## API对接

### 登录-页面获取csrf_token

superset的api真的好恶心，没有文档，只有一个swagger，所以不知道如何对接。经过摸索2天后，通过抓包后来确定了基本的对接方案。

1. 首先访问首页，获取到默认cookie和csrf_token

   ```
   curl -I 'GET' 'http://bjrdc49:8088/login/'|grep csrf
   ```

   csrf_token在返回页面的隐藏域中

   cookie在header中

   ---

   Set-Cookie: **session=eyJjc3JmX3Rva2VuIjoiNzgwOWJkZWFkYzE0N2UzYmM0Yzk3YjNmMzhhZmQ3Yzc2Y2E1MTVkZSIsImxvY2FsZSI6InpoIn0.YZzOFg.Jyvx8GWQ4AjJwFQ00yWx22Cg1W8**; HttpOnly; Path=/; SameSite=Lax

   <input id="csrf_token" name="csrf_token" type="hidden" value="Ijc4MDliZGVhZGMxNDdlM2JjNGM5N2IzZjM4YWZkN2M3NmNhNTE1ZGUi.YZzOFg.s4j0-ehzjf7Ly15Q3Uh8XfUte0Y">

   ---

2. 登录，使用获取到的cookie和csrf_token和账户密码进行登录，返回值中有一个cookie，该cookie是后续访问中需要用的cookie

   

   ```
   curl -v -X 'POST' 'http://bjrdc49:8088/login/' -H 'Cookie: session=eyJjc3JmX3Rva2VuIjoiNzgwOWJkZWFkYzE0N2UzYmM0Yzk3YjNmMzhhZmQ3Yzc2Y2E1MTVkZSIsImxvY2FsZSI6InpoIn0.YZzOFg.Jyvx8GWQ4AjJwFQ00yWx22Cg1W8' -d 'csrf_token=Ijc4MDliZGVhZGMxNDdlM2JjNGM5N2IzZjM4YWZkN2M3NmNhNTE1ZGUi.YZzOFg.s4j0-ehzjf7Ly15Q3Uh8XfUte0Y&username=admin&password=xxxxx'
   ```

   Set-Cookie: **session=.eJwtjzFuAzEMwP7iOYNk2ZaczxwkS0KCBA1wlywt-vfe0JEDAfKnbLnHcSvX9_6JS9nuXq4FwSZ0qtgbBCEIuVLUujrOkU2WIUIqEtbQRpJDs6UAglawmjylJ3BwukA6TBmDQL3BpG7WjThTovqSNNAmg5Wm8CBp1riWS1nHntv79Yivs4cFpnmoL2wcZKutyUZJoum8eCzt2D1O7_la-ozT-b6d9Dli_18qv390SELC.YZzPDw.uuUfPy3fKxdTJxCSA6rdnKitzOE**; HttpOnly; Path=/; SameSite=Lax

   ----

3. 获取chart数据

   使用登录成功后获取到的cookie获取API中的数据

   ```sh
   curl -v -X 'GET' 'http://bjrdc49:8088/api/v1/chart/256' -H 'Cookie: session=.eJwtjzFuAzEMwP7iOYNk2ZaczxwkS0KCBA1wlywt-vfe0JEDAfKnbLnHcSvX9_6JS9nuXq4FwSZ0qtgbBCEIuVLUujrOkU2WIUIqEtbQRpJDs6UAglawmjylJ3BwukA6TBmDQL3BpG7WjThTovqSNNAmg5Wm8CBp1riWS1nHntv79Yivs4cFpnmoL2wcZKutyUZJoum8eCzt2D1O7_la-ozT-b6d9Dli_18qv390SELC.YZzPDw.uuUfPy3fKxdTJxCSA6rdnKitzOE'
   ```

**注：不知为何superset的/security/login 可以登录成功，并获取到token，但是后续使用token的时候不成功**



### 登录-接口获取csrf-token

**模拟方式**

1. 获取access_token

   ```sh
   curl --location --request POST 'http://localhost:9000/api/v1/security/login' \
   --header 'Content-Type: application/json' \
   --data-raw '{
    "username": "admin",
    "password": "xxxxxx",
    "provider": "db",
    "refresh": true
   }'
   {
     "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpYXQiOjE2NTcxODYwNjEsIm5iZiI6MTY1NzE4NjA2MSwianRpIjoiNTMwYWU5NzItNzJjNi00MGQ4LWI2MzItYTU4MzhiMTkyNDc4IiwiZXhwIjoxNjU3MTg2OTYxLCJpZGVudGl0eSI6MSwiZnJlc2giOnRydWUsInR5cGUiOiJhY2Nlc3MifQ.Vj3i4tQaWdNaal2a4-PTzGCD4L5HVJtfieu9411XJio", 
     "refresh_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpYXQiOjE2NTcxODYwNjEsIm5iZiI6MTY1NzE4NjA2MSwianRpIjoiYjc5MmYwMzgtMjMzMi00M2E5LThhYzYtNzFkZjAzNDIwMDhiIiwiZXhwIjoxNjU5Nzc4MDYxLCJpZGVudGl0eSI6MSwidHlwZSI6InJlZnJlc2gifQ.Lo9mtTlyDQWHcmewg1OjAzSnhwZrGXrOaX8Oh2Mmk8E"
   }
   ```

   

2. 获取session

   authorization 为 access_token

   ```sh
   curl -i -X GET 'http://localhost:9000/api/v1/security/csrf_token/' --header 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpYXQiOjE2NTcxODYwNjEsIm5iZiI6MTY1NzE4NjA2MSwianRpIjoiNTMwYWU5NzItNzJjNi00MGQ4LWI2MzItYTU4MzhiMTkyNDc4IiwiZXhwIjoxNjU3MTg2OTYxLCJpZGVudGl0eSI6MSwiZnJlc2giOnRydWUsInR5cGUiOiJhY2Nlc3MifQ.Vj3i4tQaWdNaal2a4-PTzGCD4L5HVJtfieu9411XJio' 
   HTTP/1.1 200 OK
   X-Powered-By: Express
   content-type: application/json; charset=utf-8
   content-length: 110
   Vary: Cookie, Accept-Encoding
   set-cookie: session=eyJjc3JmX3Rva2VuIjoiYTNlYjMwZjQ1Mzg1YTNjN2JlZDUwODQ3MTE1NTZmNTRiYWNiZTYyOCJ9.Ysao8Q.3R5GG0mMYJK36YT9hcRJxhIsgYE; HttpOnly; Path=/; SameSite=Lax
   server: Werkzeug/1.0.1 Python/3.9.13
   date: Thu, 07 Jul 2022 09:35:45 GMT
   Connection: keep-alive
   Keep-Alive: timeout=5
   
   {
     "result": "ImEzZWIzMGY0NTM4NWEzYzdiZWQ1MDg0NzExNTU2ZjU0YmFjYmU2Mjgi.Ysao8A.mVwRD33d6zsTzyUNvkB3KJVZGVk"
   }
   ```

   

3. 调用接口

   ```sh
   curl -X GET 'http://localhost:9000/api/v1/chart/132' --header 'X-CSRFToken: ImEzZWIzMGY0NTM4NWEzYzdiZWQ1MDg0NzExNTU2ZjU0YmFjYmU2Mjgi.Ysao8A.mVwRD33d6zsTzyUNvkB3KJVZGVk' --header 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpYXQiOjE2NTcxODYwNjEsIm5iZiI6MTY1NzE4NjA2MSwianRpIjoiNTMwYWU5NzItNzJjNi00MGQ4LWI2MzItYTU4MzhiMTkyNDc4IiwiZXhwIjoxNjU3MTg2OTYxLCJpZGVudGl0eSI6MSwiZnJlc2giOnRydWUsInR5cGUiOiJhY2Nlc3MifQ.Vj3i4tQaWdNaal2a4-PTzGCD4L5HVJtfieu9411XJio' --header 'Cookie: session=eyJjc3JmX3Rva2VuIjoiYTNlYjMwZjQ1Mzg1YTNjN2JlZDUwODQ3MTE1NTZmNTRiYWNiZTYyOCJ9.Ysao8Q.3R5GG0mMYJK36YT9hcRJxhIsgYE'
   ```

   
   
   
