Superset
======

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



## 开发

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



### 自己开发方式（IDEA）

[参考](https://apache-superset.readthedocs.io/en/latest/installation.html)

#### 启动superset

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

#### 启动superset-frontend



#### 修改



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

superset的api真的好恶心，没有文档，只有一个swagger，所以不知道如何对接。经过摸索2天后，通过抓包后来确定了基本的对接方案。

1. 首先访问首页，获取到默认cookie和csrf_token

   ```
   curl -v -X 'GET' 'http://bjrdc49:8088/login/'
   ```

   csrf_token在返回页面的隐藏域中

   cookie在header中

   ---

   Set-Cookie: **session=eyJjc3JmX3Rva2VuIjoiNzgwOWJkZWFkYzE0N2UzYmM0Yzk3YjNmMzhhZmQ3Yzc2Y2E1MTVkZSIsImxvY2FsZSI6InpoIn0.YZzOFg.Jyvx8GWQ4AjJwFQ00yWx22Cg1W8**; HttpOnly; Path=/; SameSite=Lax

   <input id="csrf_token" name="csrf_token" type="hidden" value="Ijc4MDliZGVhZGMxNDdlM2JjNGM5N2IzZjM4YWZkN2M3NmNhNTE1ZGUi.YZzOFg.s4j0-ehzjf7Ly15Q3Uh8XfUte0Y">

   ---

2. 登录，使用获取到的cookie和csrf_token和账户密码进行登录，返回值中有一个cookie，该cookie是后续访问中需要用的cookie

   

   ```
   curl -v -X 'POST' 'http://bjrdc49:8088/login/' -H 'Cookie: session=eyJjc3JmX3Rva2VuIjoiNzgwOWJkZWFkYzE0N2UzYmM0Yzk3YjNmMzhhZmQ3Yzc2Y2E1MTVkZSIsImxvY2FsZSI6InpoIn0.YZzOFg.Jyvx8GWQ4AjJwFQ00yWx22Cg1W8' -d 'csrf_token=Ijc4MDliZGVhZGMxNDdlM2JjNGM5N2IzZjM4YWZkN2M3NmNhNTE1ZGUi.YZzOFg.s4j0-ehzjf7Ly15Q3Uh8XfUte0Y&username=admin&password=zgjx@321'
   ```

   Set-Cookie: **session=.eJwtjzFuAzEMwP7iOYNk2ZaczxwkS0KCBA1wlywt-vfe0JEDAfKnbLnHcSvX9_6JS9nuXq4FwSZ0qtgbBCEIuVLUujrOkU2WIUIqEtbQRpJDs6UAglawmjylJ3BwukA6TBmDQL3BpG7WjThTovqSNNAmg5Wm8CBp1riWS1nHntv79Yivs4cFpnmoL2wcZKutyUZJoum8eCzt2D1O7_la-ozT-b6d9Dli_18qv390SELC.YZzPDw.uuUfPy3fKxdTJxCSA6rdnKitzOE**; HttpOnly; Path=/; SameSite=Lax

   ----

3. 获取chart数据

   使用登录成功后获取到的cookie获取API中的数据

   ```sh
   curl -v -X 'GET' 'http://bjrdc49:8088/api/v1/chart/256' -H 'Cookie: session=.eJwtjzFuAzEMwP7iOYNk2ZaczxwkS0KCBA1wlywt-vfe0JEDAfKnbLnHcSvX9_6JS9nuXq4FwSZ0qtgbBCEIuVLUujrOkU2WIUIqEtbQRpJDs6UAglawmjylJ3BwukA6TBmDQL3BpG7WjThTovqSNNAmg5Wm8CBp1riWS1nHntv79Yivs4cFpnmoL2wcZKutyUZJoum8eCzt2D1O7_la-ozT-b6d9Dli_18qv390SELC.YZzPDw.uuUfPy3fKxdTJxCSA6rdnKitzOE'
   ```

**注：不知为何superset的/security/login 可以登录成功，并获取到token，但是后续使用token的时候不成功**

