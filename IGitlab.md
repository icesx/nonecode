gitlab
==============
## hosts and hostname
you must set your local's hostname is xx.net and add the correct ip with xx.net to hosts

## 错误处理

1. had an error: Acme::Client::Error::Timeout: Acme::Client::Error::Timeout

域名xx.net无法访问到，建议去掉ssl

## install

1. Install and configure the necessary dependencies

   ```
   sudo apt-get update
   sudo apt-get install -y curl openssh-server ca-certificates
   ```

   Next, install Postfix to send notification emails. If you want to use another solution to send emails please skip this step and configure an external SMTP server after GitLab has been installed.

   ```
   sudo apt-get install -y postfix
   ```

   

   During Postfix installation a configuration screen may appear. Select 'Internet Site' and press enter. Use your server's external DNS for 'mail name' and press enter. If additional screens appear, continue to press enter to accept the defaults.

2. Add the GitLab package repository and install the package
    Add the GitLab package repository.

  ```
  curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
  ```

  Next, install the GitLab package. Change https://gitlab.example.com to the URL at which you want to access your GitLab instance. Installation will automatically configure and start GitLab at that URL.

  For https:// URLs GitLab will automatically request a certificate with Let's Encrypt, which requires inbound HTTP access and a valid hostname. You can also use your own certificate or just use http://.

  ```
  sudo EXTERNAL_URL="https://gitlab.example.com" apt-get install gitlab-ee
  ```

  

3. Browse to the hostname and login
    On your first visit, you'll be redirected to a password reset screen. Provide the password for the initial administrator account and you will be redirected back to the login screen. Use the default account's username root to login.

  See our documentation for detailed instructions on installing and configuration.

4. Set up your communication preferences
    Visit our email subscription preference center to let us know when to communicate with you. We have an explicit email opt-in policy so you have complete control over what and how often we send you emails.

  Twice a month, we send out the GitLab news you need to know, including new features, integrations, docs, and behind the scenes stories from our dev teams. For critical security updates related to bugs and system performance, sign up for our dedicated security newsletter.

  IMPORTANT NOTE: If you do not opt-in to the security newsletter, you will not receive security alerts.

## 升级

> gitlab 不能跨大版本升级，其版本格式为：
>
> GitLab uses [Semantic Versioning](https://semver.org/) for its releases: `(Major).(Minor).(Patch)`.
>
> For example, for GitLab version 12.10.6:
>
> - `12` represents the major version. The major release was 12.0.0 but often referred to as 12.0.
> - `10` represents the minor version. The minor release was 12.10.0 but often referred to as 12.10.
> - `6` represents the patch number.
>
> 升级必须是从上一个`major`的最后一个版本升级到下一个`majro`的地一个版本，然后才能在一个`major`内进行小版本的升级
>
> 具体实施过程中，可以从清华的源中下载需要的版本，本地通过`dpkg`安装。
>
> 源地址为：
>
> ```http
> https://mirrors.tuna.tsinghua.edu.cn/gitlab-ee/ubuntu/pool/bionic/main/g/gitlab-ee/
> ```
>
> 

## 邮件通知

> 通过配置第三方的smtp服务来发送通知邮件，相关配置过程如下

1. 增加smtp配置，在`/etc/gitlab/gitlab.rb`中增加配置

   ```ruby
   gitlab_rails['smtp_enable'] = true
   gitlab_rails['smtp_address'] = "smtp.163.com"
   gitlab_rails['smtp_port'] = 465
   gitlab_rails['smtp_user_name'] = "13824365716"
   gitlab_rails['smtp_password'] = "password"
   gitlab_rails['smtp_domain'] = "smtp.163.com"
   gitlab_rails['smtp_authentication'] = "login"
   gitlab_rails['smtp_enable_starttls_auto'] = true
   gitlab_rails['smtp_tls'] = true
   gitlab_rails['gitlab_email_from'] = '13824365716@163.com'
   ```

2. 配置生效

   ```sh
   sudo gitlab-ctl reconfigure
   ```

3. 测试邮件

   ```
   sudo gitlab-rails consol
   ```

   在控制台中输入如下命令发送邮件

   ```ruby
   Notify.test_email('12157724@qq.com', 'Message Subject', 'Message Body').deliver_now
   ```

   正常的话会在目标邮箱12157724@qq.com中发送一条邮件，说明配置成功



## 整合kubernetes



### runner

> GitLab Runner is the open source project that is used to run your jobs and send the results back to GitLab. It is used in conjunction with [GitLab CI/CD](https://about.gitlab.com/stages-devops-lifecycle/continuous-integration/), the open-source continuous integration service included with GitLab that coordinates the jobs.

### 基本原理

runner是一个golang编写的gitlab的devops工具，用于进行ci和di工作。基本的原理是

1. gitlab中会自动生成给runner专用的token
2. runner启动前设置好gitlab的地址，和gitlab提供的token等参数，待启动后自动的会向gitlab注册自己。
3. 注册成功后，runner会接受gitlab的任务，进行诸如maven、gcc等命令的执行。
4. runner的执行，需要在项目的根目录有一个`.gitlab-ci.yml`文件来设置。
5. 该文件中会有一个`image`的属性，用于描述使用哪个docker镜像执行命令。如果是在kubernetes下的话，runner会使用kubernetes的api启动一个image的pod来执行命令。

### kubernetes下使用

#### 1. install

先安装好kubernetes，在安装成功后，在gitlab的管理界面增加kubernetes集群，相关的参数要求可以看gitlab的说明

#### 2. register

在gitlab的管理界面获取runner的token，保存下来待在kubernetes中配置的时候使用。

#### 3. kubernetes install runner

> 在kubernetes中安装runner较为复杂，网上也找不到好的资料，很多都是据一`helm`的手动安装了有一份是老外写的，但是测试有问题，只能作为参考 [老外的k8s安装runner的事例](https://edenmal.moe/post/2017/GitLab-Kubernetes-Running-CI-Runners-in-Kubernetes/#Step-3-Write-manifest-for-GitLab-CI-Runners)

经过不断的实验，后总结正确安装方式如下步骤

1. 设置namespace

   ```yaml
   cat 0-gitlab-namespace.yaml 
   apiVersion: v1
   kind: Namespace
   metadata:
     name: gitlab-runner
   ```

2. rbac

   ```yaml
   cat 1-gitlab-rbac.yaml 
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: gitlab-ci
     namespace: gitlab-runner
   ---
   kind: ClusterRole
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
   
     name: gitlab-ci
   rules:
     - apiGroups: ["", "extensions", "apps"]
       resources: ["*"]
       verbs: ["*"]
   ---
   kind: ClusterRoleBinding
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: gitlab-ci
   
   subjects:
     - kind: ServiceAccount
       name: gitlab-ci
       namespace: gitlab-runner
   roleRef:
     kind: ClusterRole
     name: gitlab-ci
     apiGroup: rbac.authorization.k8s.io
   ```
   
   `gitlab-ci`为serviceaccount，该serviceaccount需要在gitlab-runner的环境变量`KUBERNETES_SERVICE_ACCOUNT`中进行设置。
   
   `- apiGroups: ["", "extensions", "apps"]`这里的apps和extensions是api的groups，如果没有apps，可能出现如下的问题
   
   ```
   [ERROR] Failed to execute goal org.eclipse.jkube:kubernetes-maven-plugin:1.0.1:undeploy (default-cli) on project spring-cloud-k8s-provider: Execution default-cli of goal org.eclipse.jkube:kubernetes-maven-plugin:1.0.1:undeploy failed: Failure executing: DELETE at: https://172.16.15.17:6443/apis/apps/v1/namespaces/bjrdc-dev/deployments/spring-cloud-k8s-provider. Message: Forbidden!Configured service account doesn't have access. Service account may have been revoked. deployments.apps "spring-cloud-k8s-provider" is forbidden: User "system:serviceaccount:gitlab-runner:default" cannot delete resource "deployments" in API group "apps" in the namespace "bjrdc-dev". -> [Help 1]
   ```
   
   这里的`system:serviceaccount:gitlab-runner:default`是默认的serviceaccount，如果在rbac中配置了其他的serviceaccount，需要在gitlab-runner的环境变量中设置。
   
3. configmap-env 环境变量

   > 这个环境变量没有测试哪些是多余的

   ```yaml
   cat 2-gitlab-configmap-env.yaml 
   apiVersion: v1
   kind: ConfigMap
   metadata:
     labels:
       app: gitlab-ci-runner
     name: gitlab-ci-runner-cm
     namespace: gitlab-runner
   data:
     REGISTER_NON_INTERACTIVE: "true"
     REGISTER_LOCKED: "false"
     CI_SERVER_URL: "http://172.16.15.7:8090"
     METRICS_SERVER: "0.0.0.0:9100"
     RUNNER_REQUEST_CONCURRENCY: "4"
     RUNNER_EXECUTOR: "kubernetes"
     KUBERNETES_NAMESPACE: "gitlab-runner"
     KUBERNETES_SERVICE_ACCOUNT: "gitlab-ci"
     KUBERNETES_PRIVILEGED: "true"
     KUBERNETES_CPU_REQUEST: "250m"
     KUBERNETES_MEMORY_REQUEST: "256Mi"
     KUBERNETES_CPU_LIMIT: "1"
     KUBERNETES_MEMORY_LIMIT: "2Gi"
     KUBERNETES_SERVICE_CPU_REQUEST: "150m"
     KUBERNETES_SERVICE_MEMORY_REQUEST: "256Mi"
     KUBERNETES_SERVICE_CPU_LIMIT: "1"
     KUBERNETES_SERVICE_MEMORY_LIMIT: "2Gi"
     KUBERNETES_HELPER_CPU_REQUEST: "150m"
     KUBERNETES_HELPER_MEMORY_REQUEST: "100Mi"
     KUBERNETES_HELPER_CPU_LIMIT: "500m"
     KUBERNETES_HELPER_MEMORY_LIMIT: "200Mi"
     KUBERNETES_PULL_POLICY: "if-not-present"
     KUBERNETES_TERMINATIONGRACEPERIODSECONDS: "10"
     KUBERNETES_POLL_INTERVAL: "5"
     KUBERNETES_POLL_TIMEOUT: "360"
     KUBERNETES_POLL_TIMEOUT: "360"
   ```

   `KUBERNETES_SERVICE_ACCOUNT`设定的是rbac中设置的serviceaccount

4. 配置script configmap-file

   这个run.sh确实是从参考的链接里复制过来的，暂时未深入研究，但是参考链接中设置的`statefulset`和这个`run.sh`不匹配，不能一起用，我对`statefulset`进行了修改

   ```yaml
   cat 3-gitlab-configmap-file.yaml 
   apiVersion: v1
   kind: ConfigMap
   metadata:
     labels:
       app: gitlab-ci-runner
     name: gitlab-ci-runner-file
     namespace: gitlab-runner
   data:
     run.sh: |
       #!/bin/bash
       unregister() {
           kill %1
           echo "Unregistering runner ${RUNNER_NAME} ..."
           /usr/bin/gitlab-ci-multi-runner unregister -t "$(/usr/bin/gitlab-ci-multi-runner list 2>&1 | tail -n1 | awk '{print $4}' | cut -d'=' -f2)" -n ${RUNNER_NAME}
           exit $?
       }
       trap 'unregister' EXIT HUP INT QUIT PIPE TERM
       echo "Registering runner ${RUNNER_NAME} ..."
       /usr/bin/gitlab-ci-multi-runner register -r ${GITLAB_CI_TOKEN}
       #sed -i 's/^concurrent.*/concurrent = '"${RUNNER_REQUEST_CONCURRENCY}"'/' /home/gitlab-runner/.gitlab-runner/config.toml
       echo "Starting runner ${RUNNER_NAME} ..."
       /usr/bin/gitlab-ci-multi-runner run -n ${RUNNER_NAME} &
       wait
   ```

5. 设置runner token

   ```yaml
   cat 4-gitlab-secret-token.yaml 
   apiVersion: v1
   kind: Secret
   metadata:
     name: gitlab-ci-token
     namespace: gitlab-runner
     labels:
       app: gitlab-ci-runner
   data:
     GITLAB_CI_TOKEN: eURCRkg1RXFIaVF4elFSeDg2V0UK
   ```

   如上的`eURCRkg1RXFIaVF4elFSeDg2V0UK`是通过将gitlab提供的token base64后获得

   ```sh
   echo token | base64 -w0
   ```

6. 设置statefulset

   ```yaml
   cat 5-gitlab-runner-stateful.yaml 
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: gitlab-ci-runner
     namespace: gitlab-runner
     labels:
       app: gitlab-ci-runner
   spec:
     selector:
       matchLabels:
         app: gitlab-ci-runner
     updateStrategy:
       type: RollingUpdate
     replicas: 2
     serviceName: gitlab-ci-runner
     template:
       metadata:
         labels:
           app: gitlab-ci-runner
       spec:
         hostAliases:
         - ip: "172.16.15.7"
           hostnames:
           - "bjrdc7"
         affinity:
           podAntiAffinity:
             requiredDuringSchedulingIgnoredDuringExecution:
               - topologyKey: "kubernetes.io/hostname"
                 labelSelector:
                   matchExpressions:
                   - key: app
                     operator: In
                     values:
                     - gitlab-ci-runner
         volumes:
         - name: gitlab-ci-runner-file
           configMap:
             name: gitlab-ci-runner-file
             defaultMode: 0755
         serviceAccountName: gitlab-ci
         containers:
         - image: gitlab/gitlab-runner:v12.6.0
           name: gitlab-ci-runner
           command:
           - /scripts/run.sh
           envFrom:
           - configMapRef:
               name: gitlab-ci-runner-cm
           - secretRef:
               name: gitlab-ci-token
           env:
           - name: RUNNER_NAME
             valueFrom:
               fieldRef:
                 fieldPath: metadata.name
           - name: RUNNER_PRE_CLONE_SCRIPT
             value: "echo '172.16.15.7 bjrdc7' >> /etc/hosts"
           - name: RUNNER_PRE_BUILD_SCRIPT
             value: for i in {1..254}; do echo 172.16.15.$i bjrdc$i >> /etc/hosts;done
           ports:
           - containerPort: 9100
             name: http-metrics
             protocol: TCP
           volumeMounts:
           - name: gitlab-ci-runner-file
             mountPath: /scripts/run.sh
             subPath: run.sh
             readOnly: true
         restartPolicy: Always
   
   ```

   原有的样例中是非root执行，会有一个缺少config.toml的坑

   ```yaml
           - name: RUNNER_PRE_CLONE_SCRIPT
             value: "echo '172.16.15.7 bjrdc7' >> /etc/hosts"
           - name: RUNNER_PRE_BUILD_SCRIPT
             value: for i in {1..254}; do echo 172.16.15.$i bjrdc$i >> /etc/hosts;done
   ```

   如上代码用于增加runner和maven的pod中需要域名

7. .gitlab-ci.yml

   ```yaml
   image: maven:3.6.3-jdk-8
   variables:
      MAVEN_CLI_OPTS: -s settings.xml --batch-mode
      MAVEN_OPTS: -DskipTests=false
   cache:
      paths:
      - .m2/repository/
      - target/
   build:
      stage: build
      script:
      - mvn $MAVEN_CLI_OPTS compile
   test:
      stage: test
      script:
      - mvn $MAVEN_CLI_OPTS test $MAVEN_OPTS
   deploy:
      stage: deploy
      script:
      - mvn $MAVEN_CLI_OPTS deploy 
      only:
      - master
   ```

8. settings.xml

   需要将settings.xml放到项目根目录和`.gitlab-ci.yml`相同目录

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
   	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   	xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
   
   	<localRepository>.m2/repository</localRepository>
   	<pluginGroups>
   	</pluginGroups>
   	<proxies>
   	</proxies>
   
   	<servers>
   		<server>
   			<id>nexus</id>
   			<username>${env.MAVEN_REPO_USER}</username>
   			<password>${env.MAVEN_REPO_PASS}</password>
   		</server>
   	</servers>
   	<mirrors>
   		<mirror>
   			<id>nexus</id>
   			<mirrorOf>*</mirrorOf>
   			<name>nexus component</name>
   			<url>http://bjrdc9:8099/repository/maven-public</url>
   		</mirror>
   	</mirrors>
   
   	<profiles>
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
   </settings>
   ```

9. 在gitlab管理后台中增加环境变量`MAVEN_REPO_PASS`和`MAVEN_REPO_USER`用于登录私有仓库，该两个变量最终会兑现到`settings.xml`

## kubernetes-maven-plugin 插件

使用kubernetes-maven-plugin插件进行项目打包可以将项目打包到docker，并部署到kubernetes中。

`<dockerHost>http://bjrdc218:2375</dockerHost>`：远程docker的地址。

`<registry>${harbor.local}</registry>`: redistry 地址

`<dockerFile>${project.basedir}/src/main/docker/Dockerfile</dockerFile>`：打包的dockerFile路径

`<masterUrl>${k8s.master.url}</masterUrl>`：kubernetes地址

注：需要本地主机有`.kube/config`文件，用于访问kubernetes集群。否则需要在`access`中配置访问集群的帐号密码等。

```xml
				<plugin>
					<groupId>org.eclipse.jkube</groupId>
					<artifactId>kubernetes-maven-plugin</artifactId>
					<version>1.0.1</version>
					<configuration>
						<skip>true</skip>
						<dockerHost>http://bjrdc218:2375</dockerHost>
						<registry>${harbor.local}</registry>
						<authConfig>
							<username>admin</username>
							<password>Harbor12345</password>
						</authConfig>
						<namespace>bjrdc-dev</namespace>
						<kubernetesManifest>${project.build.directory}/jkube/kubernetes.yml</kubernetesManifest>
						<targetDir>${project.build.directory}/jkube</targetDir>
						<images>
							<image>
								<registry>${harbor.local}</registry>
								<name>bjrdc-dev/${project.artifactId}:${project.version}</name>
								<alias>${project.artifactId}</alias>
								<build>
									<assembly>
										<name>${k8s.plugin.assemble.name}</name>
									</assembly>
									<contextDir>${project.basedir}</contextDir>
									<dockerFile>${project.basedir}/src/main/docker/Dockerfile</dockerFile>
									<!-- <cleanup>remove</cleanup> -->
								</build>
							</image>
						</images>
						<access>
							<masterUrl>${k8s.master.url}</masterUrl>
						</access>
					</configuration>
				</plugin>
				<plugin>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-maven-plugin</artifactId>
					<version>${spring-boot.version}</version>
					<configuration>
						<mainClass>${start-class}</mainClass>
					</configuration>
				</plugin>
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
							<JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
						</buildArgs>
					</configuration>
				</plugin>
```

执行

```shell
mvn clean package spring-boot:repackage  k8s:build  k8s:resource k8s:push k8s:undeploy k8s:deploy
```

## 远程docker

参考

[docker]: IDocker.md

## 问题处理

##### fatal: unable to access https://git.exemple.io: Could not resolve host

是因为kubernet启动的maven的pod没有git的host通过环境变量增加即可

```yaml
        - name: RUNNER_PRE_CLONE_SCRIPT
          value: "echo '172.16.15.7 bjrdc7' >> /etc/hosts"
```

##### [ERROR] The specified user settings file does not exist: /git/.m2/settings.xml

就是因为没有这个文件了，这个文件需要mount进去，然后用`.gitlab-ci.yml`指定就可以了

#### pending

gitlab中的任务一直处于pending中



## 备份

### 基本命令

gitlab提供的备份命令为gitlab-rake，备份命令使用如下:

```shell
gitlab-rake gitlab:backup:create
```

该命令会备份gitlab仓库、数据库、用户、用户组、用户密钥、权限等信息。

备份完成后备份文件会出现在`/var/opt/gitlab/backups/`

**该目录的权限必须是**，否则会有错误`Errno::EACCES: Permission denied - /git/backups/gitlab/db/database.sql.gz`

```
drwxr-xr-x  5 git   git   4096 Dec 14 22:00 gitlab
```



### 恢复命令

`1537261122_2018_09_18_9.2.5`对应的`1607369941_2020_12_08_13.3.6-ee_gitlab_backup.tar`

```sh
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq
gitlab-rake gitlab:backup:restore BACKUP=1607959169_2020_12_14_13.3.6-ee
gitlab-ctl start
```



### 修改备份目录

vim /etc/gitlab/gitlab.rb

修改上图`backup_path`的值即可，之后使用`gitlab-ctl reconfigure`使得配置生效

### 一个备份脚本

```sh
cat gitlab-backup.sh 
set -x
workdir=/backups/gitlab/
rmfile=`su - git -c "ls -t $workdir|grep backup.tar|tail -1"`
if [ ! -n "$rmfile" ];then
        echo "empty"
else
        echo $rmfile
        su - git -c "rm -f $workdir/$rmfile"
fi
/opt/gitlab/bin/gitlab-rake gitlab:backup:create
```



## 版本管理

> 通过gitlab CI/CD maven nexus进行自动化开发的时候，需要进行版本管理，防止在发布或者开发过程中，nexus中的版本被人修改。

### 基本原则

1. snapshot版本是开发人员可以自动进行发布的。
2. release版本只能通过gitlab进行发布，开发人员没有release版本发布的密码。
3. 版本号格式`major.minor.version`-`[RELEASE|SNAPSHOT]`
   1. major：如果去掉接口、类或者功能的版本，需要对major+1
   2. minor：如果增加接口，但是保持向下兼容，需要minor+1
   3. version：修改bug后，发布的版本version+1。`一次发布version+1，不是修改一个bug就+1`
4. 每个项目的master分支的版本必须是release版本，版本的规范为`major.minor.version-RELEASE`
5. 每个项目的开发分支的版本好可以自定义如`dev`、`dev-xxx`等
6. 每个项目需要发布的带版本的项目必须是``major.minor.version`-`[RELEASE|SNAPSHOT]`
7. release版本只能通过master发布，并且只能通过gitlab发布，snapshot可以手动或者gitlab发表。
8. 正式发布的版本最好依赖release版本的，release版本一般不更新，但凡有更新，需要版本好变化，如此可以保证老系统的未定性。
9. snapshot的版本，可以由开发人员自行发布，没有太多限制。
10. snapshot版本稳定后，需要merge到master分支，并发布master分支新版本。

### 开发流程

1. 在gitlab上创建项目后，创建`0.0.1-SNAPSHOT`分支，如此项目将有两个分支`0.0.1-SNAPSHOT`和`master`

2. master分支：在master分支下创建`.gitlab-ci.yml`文件，内容如下。

   > 告知gitlab按照如下build/test/deploy过程进行自动构建。如果不需要deploy可以删除depoly节点

   ```yaml
   image: maven:3.6.3-jdk-8
   variables:
      MAVEN_CLI_OPTS: -s settings.xml --batch-mode
      MAVEN_OPTS: -DskipTests=false
   cache:
      paths:
      - .m2/repository/
      - target/
   build:
      stage: build
      script:
      - mvn $MAVEN_CLI_OPTS compile
   test:
      stage: test
      script:
      - mvn $MAVEN_CLI_OPTS test $MAVEN_OPTS
   deploy:
      stage: deploy
      script:
      - mvn $MAVEN_CLI_OPTS -DskipTests=true deploy
      only:
      - 0.0.1-SNAPSHOT
   ```

3. `0.0.1-SNAPSHOT`分支下增加`.gitlab-ci.yml`文件，内容如下

   ```
   image: maven:3.6.3-jdk-8
   variables:
      MAVEN_CLI_OPTS: -s settings.xml --batch-mode
      MAVEN_OPTS: -DskipTests=false
   cache:
      paths:
      - .m2/repository/
      - target/
   build:
      stage: build
      script:
      - mvn $MAVEN_CLI_OPTS compile
   test:
      stage: test
      script:
      - mvn $MAVEN_CLI_OPTS test $MAVEN_OPTS
   deploy:
      stage: deploy
      script:
      - mvn $MAVEN_CLI_OPTS -DskipTests=true deploy
      only:
      - 0.0.1-SNAPSHOT
   ```

4. 在两个分支下都增加`settings.xml`文件，内容如下

   > 告诉maven按照如下配置进行自动的打包和deploy

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
   	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   	xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
   	<localRepository>.m2/repository</localRepository>
   	<pluginGroups>
   	</pluginGroups>
   	<proxies>
   	</proxies>
   	<servers>
   		<server>
   			<id>nexus</id>
   			<username>${env.MAVEN_REPO_USER}</username>
   			<password>${env.MAVEN_REPO_PASS}</password>
   		</server>
   		<server>
   			<id>release</id>
   			<username>${env.MAVEN_RELEASE_USER}</username>
   			<password>${env.MAVEN_RELEASE_PASS}</password>
   		</server>
   	</servers>
   	<mirrors>
   		<mirror>
   			<id>nexus</id>
   			<mirrorOf>*</mirrorOf>
   			<name>nexus component</name>
   			<url>http://bjrdc9:8099/repository/maven-public</url>
   		</mirror>
   	</mirrors>
   	<profiles>
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
   </settings>
   ```

5. 如果有需要可以在创建其他开发分支，用于临时开发，开发分支一旦合并到`NAPSHOT`支后需要删除不需要的开发分支。

6. 如有必要可以增加`SNAPSHOT`分支，最多保留3个`SNAPSHOT`。

7. `SNAPSHOT`分支稳定后，通过在gitlab上提交`merge request`来申请将`SNAPTSHOT`分支合并到master分支，作为release版本。

8. `release`分支测试完成并发布后，进行打TAG的操作。

### 注意点

1. 链接库如common、api等最好是依赖到`RELEASE`分支。
2. 如果要将`SNAPSHOT`中的功能发布到`RELEASE`需要进行`merge request`
3. 分支规范需要按照`major.minor.version`-`[RELEASE|SNAPSHOT]`，临时分支可以不按照次规范。