Jmeter
======

## case编写

### 线程组



### 控制器



### csv变量

1. 原始文件

   ```
   id	name	mobile_phone	uuid	reg_id	user_type	
   38	name1	15574923125	866146032475505,866146032475513	7e74b532451b67975fd8811bec8b643b	1	
   46	name2	15227125901	514c4fa924af8143	67661774a4760402fa2a9e3399412842	4
   47	15611186149	15611186149	867367032098712,867367032098704	50ea4ea519afae41fbf98357fd1d7faa	1
   ```

2. 添加 CSV Data Set Config

   

3. sample->http request->paramters

   userid	${id}

   name	${name}

### test fragment

> 用于批量增加fragment



## 结果处理



### 返回json取值

1. 在查看结果树的地方可以设置通过“Json Path Tester”查看结果，在Json Path Tester的“Json Path Expression”输入表达式"$.aUser.id"，可以获取到aUser对象的Id属性。
2. 为当前的Http Request增加一个后置处理器“Json Extractor”，其中
   1. Name of Create Variable 中输入变量名user_id
   2. Json Path expression 中输入 $.aUser.id
3. 如此在其他的http Request中即可通过${user_id}获取到当前请求返回值

### cookies

 1. The cookie manager stores and sends cookies just like a web browser.

    If you have an HTTP Request and the response contains a cookie, the Cookie Manager automatically stores that cookie and will use it for all future requests to that particular web site. Each JMeter thread has its own "cookie storage area". So, if you are testing a web site that uses a cookie for storing information for particular sessions then each JMeter thread will have its own session. **Note that such cookies do not appear on the Cookie Manager display, but they can be seen using the View Results Tree Listener.

 2. 

## 函数

### 日期函数

#### time

1. ${__time(dd/MM/yyyy,)}

#### timeShift

1. ${__timeShift(yyyy-MM-dd HH:mm:ss,,P2D,,)} will generate a future date by adding 2 days in the current day
2. ${__timeShift(yyyy-MM-dd HH:mm:ss,,PT2H,,)} will generate a future time by adding 2 hours in the current time
3. ${__timeShift(yyyy-MM-dd HH:mm:ss,,PT2M,,)} will generate a future time by adding 2 minutes in the current time
4. ${__timeShift(yyyy-MM-dd HH:mm:ss,,P2DT2H,,)} will generate a future date and time by adding 2 days and 2 hours in the current timestamp
5. ${__timeShift(yyyy-MM-dd HH:mm:ss,,-PT2M,,)} will generate a past time by reducing 2 minutes in the current time
6. ${__timeShift(yyyy-MM-dd HH:mm:ss,,-P2DT2H,,)} will generate a past date and time by reducing 2 days and 2 hours in the current timestamp
7. ${__timeShift(yyyy-MM-dd HH:mm:ss,2020-10-20,P2D,,)} will generate 2020-10-22 which is the next 2nd day from the specified date.