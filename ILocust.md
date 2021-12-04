Locust
======

> Locust is an easy-to-use, distributed, user load testing tool. It is intended for load-testing web sites (or other systems) and figuring out how many concurrent users a system can handle.

## install

> 使用virtualenv安装

```
virtualenv --no-site-packages --python=python3.6 venv
source ./venv/bin/activate
pip3 install locust
locust --help
```



## Use

### 简单脚本

> 编辑测试脚本
>
> ```
> cat >locust_restful.py <<EOF
> import random
> from locust import HttpUser, task, between
> 
> class QuickstartUser(HttpUser):
>     wait_time = between(5, 9)
> 
>     @task
>     def index_page(self):
>         self.client.get("/sc-k8s-consumer/feign/list")
>         self.client.get("/sc-k8s-consumer/ribbon/list")
>     
>     def on_start(self):
>         pass 
> EOF        
> ```
>
> 运行脚本
>
> ```
> source ~/venv/bin/activate
> locust -f locust_restful.py
> ```
>
> 在浏览器端打开网页`bjrdc8:8089`输入访问的地址，用户数等参数
>
> 注：访问地址需要带http

### 复杂脚本

> 官方有一个带登录的脚本，详细如下
>
> ```
> cat >locustfile_login.py <<EOF
> import random
> from locust import HttpUser, task, between
> 
> class QuickstartUser(HttpUser):
>     wait_time = between(5, 9)
> 
>     @task
>     def index_page(self):
>         self.client.get("/hello")
>         self.client.get("/world")
> 
>     @task(3)
>     def view_item(self):
>         item_id = random.randint(1, 10000)
>         self.client.get(f"/item?id={item_id}", name="/item")
> 
>     def on_start(self):
>         self.client.post("/login", {"username":"foo", "password":"bar"})
>         
> EOF        
> ```
>
> 