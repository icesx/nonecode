Maven
========

### 基本命令



### goal

> 将当前插件的resource、build、deploy阶段 绑定到deploy阶段

```xml
<execution>
    <phase>deploy</phase>
    <id>default</id>
    <goals>
        <goal>resource</goal>
        <goal>build</goal>
        <goal>deploy</goal>
    </goals>
</execution>
```



### phase



| `process-resources`      | `resources:resources`     |
| ------------------------ | ------------------------- |
| `compile`                | `compiler:compile`        |
| `process-test-resources` | `resources:testResources` |
| `test-compile`           | `compiler:testCompile`    |
| `test`                   | `surefire:test`           |
| `package`                | `jar:jar`                 |
| `install`                | `install:install`         |
| `deploy`                 | `deploy:deploy`           |

## 自定义插件



```xml
<execution>
	<phase>process-sources</phase>
    <goals>
    	<goal>modifyCode</goal>
    </goals>
</execution>
```

modifyCode 将关联到MavenModleCodeModifyPlugin

```java
@Mojo(name = "modifyCode", requiresDependencyResolution = ResolutionScope.COMPILE)
public class MavenModleCodeModifyPlugin extends AbstractMojo {
...
	@Override
	public void execute() throws MojoExecutionException, MojoFailureException {
	}
}
```



# nexus about 

### 安装
1. 官网下载最新版本，解压缩
	https://www.sonatype.com/download-oss-sonatype
2. 修改端口
```
vi ${nexus_home}/etc/nexus-default.properties
```
### 密码修改
1. 停服

2. 进入OrientDB控制台
	
```
cd ${nexus_home}
	java -jar ./lib/support/nexus-orient-console.jar
```

3. 在控制台执行：
```
  connect plocal:/home/bjrdc/software/sonatype-work/nexus3/db/security admin admin
```
4. 重置密码为admin123
```
update user SET password="$shiro1$SHA-512$1024$NE+wqQq/TmjZMvfI7ENh/g==$V4yPw8T64UQ6GfJfxYq2hLsVrBY8D1v+bktfOxGdt4b/9BthpWPNUy/CBk6V9iA0nHpzYzJFWO8v/tZFtES8CA==" UPSERT WHERE id="admin"
```
5. 启动服务

```
./nexus start
```

###  添加阿里源
在nexus后台添加repository aliyun-public
http://maven.aliyun.com/nexus/content/groups/public

### maven-public
在maven-public repository中增加 aliyun-public 

# maven about
### release 和 snapshot的管理

1. 在maven中增加release的帐号，并将帐号增加到settings中

   ```xml
   <server>
   	<id>nexus</id>
   	<username>deploy</username>
   	<password>xxxx</password>
   </server>
     <server>
   	  <id>release</id>
   	  <username>release</username>
       <password>bjrdc!@#321!</password>
     </server>
   ```

2. 在项目的pom.xml中增加发布地址

   ```xml
   	<distributionManagement>
   		<snapshotRepository>
   			<id>nexus</id>
   			<url>http://bjrdc9:8099/repository/maven-snapshots</url>
   		</snapshotRepository>
   		<repository>
   			<id>release</id>
   			<name>release</name>
   			<url>http://bjrdc9:8099/repository/maven-releases</url>
   		</repository>
   	</distributionManagement>
   ```

   

3. 通过如下命令部署到nexus中

   ```
   mvn deploy
   ```

   > 注：如果是SNAPSHOT版本，则自动上传到SNAPSHOT仓库中。如果是release版本，则自动上传到release版本中。

### maven/conf/settings 

```xml
<servers>
<server>
	<id>nexus</id>
	<username>deploy</username>
	<password>xxxx</password>
</server>
</servers>
....
<mirror>
    <id>nexus</id>
    <!--nexus must = server.nexus-->
    <mirrorOf>*</mirrorOf>
    <name>nexus component</name>
    <url>http://bjrdc9:8099/repository/maven-public</url>
</mirror>
<profile>  
      <id>nexus</id>
      <repositories>
        <repository>
          <id>nexus</id>
          <url>http://bjrdc9:8099/repository/maven-public</url>
          <releases>
            <enabled>true</enabled>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </snapshots>
        </repository>
      </repositories>
      <activation>
        <activeByDefault>true</activeByDefault>      
        <jdk>1.8</jdk>
      </activation>
      <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
      </properties>
    </profile>
  </profiles>
<activeProfiles>
    <activeProfile>nexus</activeProfile>
</activeProfiles>
```


## 本地代码deploy
在项目的pom文件中增加如下配置
```xml
	<distributionManagement>
		<snapshotRepository>
			<id>nexus</id>
			<!--等于mirror中server.nexus-->
			<url>http://bjrdc9:8099/repository/maven-snapshots</url>
		</snapshotRepository>
	</distributionManagement>
	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.7.0</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-resources-plugin</artifactId>
				<configuration>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-source-plugin</artifactId>
				<version>3.0.1</version>
				<executions>
					<execution>
						<id>attach-sources</id>
						<goals>
							<goal>jar</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-deploy-plugin</artifactId>
				<version>2.8.2</version>
				<executions>
					<execution>
						<id>deploy</id>
						<phase>deploy</phase>
						<goals>
							<goal>deploy</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
```

## 本地第三方库提交

