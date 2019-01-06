Spring-boot
===================
Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run".

We take an opinionated view of the Spring platform and third-party libraries so you can get started with minimum fuss. Most Spring Boot applications need very little Spring configuration

###init
https://start.spring.io/

###已发现的特性
+ 自动装配
	根据classpath中的依赖判断当前项目的特点
+ 调试简单
+ docker集成
+ 嵌入式Tomcat
+ 

### War
mvn clean package
###actuator
####依赖
```
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
			<optional>true</optional>
		</dependency>

```
####Endpoint：

+ /autoconfig，用来查看Spring Boot的框架自动配置信息，哪些被自动配置，哪些没有，原因是什么。
+ /beans，显示应用上下文的Bean列表
+ /dump，显示线程dump信息
+ /health，应用健康状况检查
+ /metrics
+ /shutdown, 默认没有打开
+ /trace


###Spring-cloud

