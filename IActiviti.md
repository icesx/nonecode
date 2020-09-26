activiti
=========

### eclipse

``` http
http://activiti.org/designer/update/
https://www.activiti.org/designer/archived/
/ICESX/workSpaceMoa/ministry-of-agriculture/doc/pig-transport/resources/eclipse-plugin/activiti-designer-5.18.0.zip
```

> 不显示属性卡的时候，打开eclipse的properties view

## 一些基本概念

1. bpmn：activiti的流程声明文件，用户可以自定义该文件，用于声明流程
2. 流程实例：使用activiti引擎和activiti key 
3. depoly:

### deploy

将 bpmn文件部署到数据库

```java
	@Override
	public Deployment deploy(String classpath) {
		return repositoryService.createDeployment().addClasspathResource(classpath).deploy();

	}
```

### 权限

activiti 使用候选人和候选组的概念进行执行，提前需要通过

#### Candidate 候选人

一般用于实现自动领取任务

#### assignee 代理人

通过代理人执行：assignee

```java
@Test
public void claimTask() {
  // 上面查询到user1的候选任务id为2505
  String taskId = "2505";
  // 将2505任务分配给user1处理
  taskService.claim(taskId, "user1");
}
```

### 参数



## 架构

如何通过剥离activiti与业务流程，以达到在业务流程变化的时候，代码不作修改呢？



## 问题处理

### spring-boot2

```java
Caused by: java.lang.ArrayStoreException: sun.reflect.annotation.TypeNotPresentExceptionProxy
 
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'requestMappingHandlerMapping' defined in class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]: Invocation of init method failed; nested exception is java.lang.ArrayStoreException: sun.reflect.annotation.TypeNotPresentExceptionProxy

```

```java
@SpringBootApplication(exclude = SecurityAutoConfiguration.class)
```



