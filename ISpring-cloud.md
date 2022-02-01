Spring-cloud
===

目前尝试了两种方式

> spring-cloud 原生
>
> 使用spring-cloud的微服务的组件，进行搭建。这些微服务的组件可以在docker或者k8s上执行，不使用k8s的configmap或者服务发现机制。



> k8s原生
>
> 使用spring-cloud-kubernetes组件实现spring-cloud与k8s的深度整合。

## Spring-cloud native

### on docker

pom.xml中增加如下配置

```
<build>
<pluginManagement>
			<plugins>
				<plugin>
					<groupId>com.spotify</groupId>
					<artifactId>dockerfile-maven-plugin</artifactId>
					<version>1.4.13</version>
					<executions>
						<execution>
							<id>default</id>
							<goals>
								<goal>build</goal>
								<goal>push</goal>
							</goals>
						</execution>
					</executions>
					<configuration>
						<skip>true</skip>
						<repository>bjrdc206.reg/bjrdc-dev/${project.artifactId}</repository>
						<tag>${project.version}</tag>
						<buildArgs>
							<JAR_FILE>${project.build.finalName}.jar</JAR_FILE>
						</buildArgs>
					</configuration>
				</plugin>
			</plugins>
		</pluginManagement>
	</build>
```

在项目根目录创建Dockerfile，内容如下


```
FROM java:8-jdk
ARG JAR_FILE
ADD target/${JAR_FILE} app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```
本地安装好docker，并配置docker为not sudo可以执行

通过如下命令push镜像到docker

```
mvn package dockerfile:push
```



### on k8s

