flask
===

### Blueprint

### flask_appbuilder

Simple and rapid application development framework, built on top of [Flask](http://flask.pocoo.org/). Includes detailed security, auto CRUD generation for your models, google charts and much more.

https://flask-appbuilder.readthedocs.io/en/latest/installation.html

#### install

```
$ pip install flask-appbuilder
```

## 启动原理

### @main.app_context_processor

1. app_context_processor作为一个装饰器修饰一个函数。
2. 函数的返回结果必须是dict，届时dict中的key将作为变量在所有模板中可见。模版中中进行使用`邮箱:<a href="mailto:{{email}}">{{email}}</a>`

### @app.shell_context_processor

### @with_appcontext

[`@with_appcontext`](http://flask.pocoo.org/docs/1.0/api/#flask.cli.with_appcontext) decorator to ensure the command runs in an app context.
