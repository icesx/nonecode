Spring-boot
===================
Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run".

We take an opinionated view of the Spring platform and third-party libraries so you can get started with minimum fuss. Most Spring Boot applications need very little Spring configuration

### init
https://start.spring.io/

### 已发现的特性

+ 自动装配
	根据classpath中的依赖判断当前项目的特点
+ 调试简单
+ docker集成
+ 嵌入式Tomcat
+ 

### War
mvn clean package
### actuator
#### 依赖

```
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
			<optional>true</optional>
		</dependency>

```
#### Endpoint：

+ /autoconfig，用来查看Spring Boot的框架自动配置信息，哪些被自动配置，哪些没有，原因是什么。
+ /beans，显示应用上下文的Bean列表
+ /dump，显示线程dump信息
+ /health，应用健康状况检查
+ /metrics
+ /shutdown, 默认没有打开
+ /trace

### Classloader

1. spring-boot中使用较多的类加载器 
   1.  在使用eclipse调试或者执行的时候，使用的是RestartClassLoader 
   2. 在使用java -jar 执行的时候，使用的是LaunchedURLClassLoader
2. RestartClassLoader 
   1.  .class文件被修改后自动重新启动
   2.  不加载addUrl到classpath中的jar文件，不会加载 
   3. 所有如果需要动态加载jar的项目，则无法实现，可以使用变通的方式，就是在重启的时候在RestartClassLoader之前将jar文件加载

```
@SpringBootApplication
@ComponentScan({ "com.xjgz.component.test.springboot.ctrl", "com.xjgz.component.test.springboot.service" })
public class Application {
    private static final Logger logger = LoggerFactory.getLogger(Application.class);
    public static void main(String[] args) {
        logger.info("classloader {}", CurrentClassLoader.currentLoader().toString());
        ClasspathManager.instance().reload();
        SpringApplication.run(Application.class, args);
    }
}
```

### Spring-cloud

