# 1、代码服务器搭建

- Jenkins，原名Hudson，2011 年改为现在的名字。它是一个开源的实现持续集成的软件工具；


- 官方网站：https://www.jenkins.io/


## 1.1、GitLab安装使用

- 官方网站：https://about.gitlab.com/


- 安装所需最小配置：内存至少4G，https://docs.gitlab.cn/jh/install/requirements.html


## 1.2、在ssh下安装

- 官方安装文档：https://gitlab.cn/install/?version=ce


### 1.2.1、安装依赖

```
sudo yum install -y curl policycoreutils-python openssh-server perl
sudo systemctl enable sshd
sudo systemctl start sshd
```

### 1.2.2、配置镜像

```
curl -fsSL https://packages.gitlab.cn/repository/raw/scripts/setup.sh | /bin/bash
```

### 1.2.3、开始安装

```
sudo EXTERNAL_URL="http://192.168.44.103" yum install -y gitlab-jh
```

- 除非您在安装过程中指定了自定义密码，否则将随机生成一个密码并存储在 `/etc/gitlab/initial_root_password` 文件中。(出于安全原因，24 小时后，此文件会被第一次 `gitlab-ctl reconfigure` 自动删除，因此若使用随机密码登录，建议安装成功初始登录成功之后，立即修改初始密码）。使用此密码和用户名 `root` 登录；

  ```shell
  vim /etc/gitlab/initial_root_password
  ```

  ![image-20221123080959150](./assets/image-20221123080959150.png)

  ![image-20221122220808228](./assets/image-20221122220808228.png)

- gitlab常用命令：

  ```shell
  #启动所有gitlab组件
  gitlab-ctl start                  
  #停止所有gitlab组件
  gitlab-ctl stop                   
  #重启所有gitlab组件
  gitlab-ctl restart                
  #查看服务状态
  gitlab-ctl status                 
  #启动服务
  gitlab-ctl reconfigure            
  #修改默认的配置文件
  vi /etc/gitlab/gitlab.rb         
  #查看日志
  gitlab-ctl tail                   
  ```

- 开放端口：https://blog.csdn.net/weixin_51687288/article/details/121415885

  ```shell
  #开启防火墙
  service firewalld start
  #添加http服务到firewalld,pemmanent表示永久生效，若不加–permanent系统下次启动后就会失效
  sudo firewall-cmd --permanent --add-service=http
  #重启防火墙
  sudo systemctl reload firewalld
  #查看端口号是否已开
  firewall-cmd --query-port=8099/tcp
  #添加指定需要开放的端口
  firewall-cmd --add-port=8099/tcp --permanent
  #重载入添加的端口
  firewall-cmd --reload
  #查询指定端口是否开启成功
  firewall-cmd --query-port=8099/tcp
  ```

## 1.3、在Docker下安装

- 官方安装文档：https://docs.gitlab.cn/jh/install/docker.html


- 安装所需最小配置，内存至少4G，系统内核至少在3.10以上 `uname -r` 命令可查看系统内核版本;


### 1.3.1、安装Docker

```shell
#更新yum源
yum update
#安装依赖
yum install -y yum-utils device-mapper-persistent-data lvm2
#添加镜像
#国外镜像
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
#阿里镜像
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#查看源中可使用版本
yum list docker-ce --showduplicates | sort -r
#安装指定版本
yum install docker
yum install docker-ce-20.10.9-3.el7
#配置开机启动项
systemctl start docker
systemctl enable docker
docker version
```

### 1.3.2、使用容器安装gitlab

```shell
#添加容器，hostname需要更改
docker run --detach \
  --hostname 192.168.182.102 \
  --publish 443:443 --publish 80:80 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab:Z \
  --volume $GITLAB_HOME/logs:/var/log/gitlab:Z \
  --volume $GITLAB_HOME/data:/var/opt/gitlab:Z \
  --shm-size 256m \
  registry.gitlab.cn/omnibus/gitlab-jh:latest
#启动容器
docker start gitlab
#查看已存在的容器
docker ps -a
#进入容器
docker exec -it  gitlab /bin/bash
```

### 1.3.3、访问登录

- 当首次运行出现502错误的时候排查两个原因：

  - 虚拟机内存至少需要4G，`free -m`可查看；

  - 稍微等待，然后刷新；


- #### 管理账号登录

  - 用户名：root
  - 密码：存在`docker exec -it  gitlab /bin/bash`中，登录后需要改密码，否则24小时之后会自动失效；

### 1.3.4、创建空白项目

![image-20221123210511213](./assets/image-20221123210511213.png)

# 2、Jenkins服务器搭建

- 官方文档：https://www.jenkins.io


- 安装需求：
  - 机器要求：256MB内存，建议大于512MB、10GB硬盘（用于Jenkins和Docker镜像）；
  - 软件要求：Java8（JRE或者JDK都可以）、Docker；

## 2.1、安装JDK

```shell
#检索可用包
yum search java|grep jdk
#这个只是装了JRE
yum install java-1.8.0-openjdk
#默认yum安装java的时候会显示安装的是openjdk1.8 实则实际上只安装了jre
yum install -y java-devel
#安装目录
cd /usr/lib/jvm
#开启防火墙
service firewalld start
#添加http服务到firewalld,pemmanent表示永久生效，若不加–permanent系统下次启动后就会失效
sudo firewall-cmd --permanent --add-service=http
#重启防火墙
sudo systemctl reload firewalld
#查看端口号是否已开
firewall-cmd --query-port=8080/tcp
#添加指定需要开放的端口
firewall-cmd --add-port=8080/tcp --permanent
#重载入添加的端口
firewall-cmd --reload
#查询指定端口是否开启成功
firewall-cmd --query-port=8080/tcp
```

JAVA_HOME配置(可选)

```shell
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.322.b06-1.el7_9.x86_64

export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.322.b06-1.el7_9.x86_64
export JRE_HOME=$JAVA_HOME/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH

jdk
/etc/alternatives/jre_openjdk
source /etc/profile 立即生效
```

## 2.2、安装maven

```shell
tar -zxvf maven压缩包
mv maven文件夹 /usr/local/maven
/usr/local/maven/bin/mvn
/usr/local/maven/bin/mvn -v
#测试maven打包
/usr/local/maven/bin/mvn clean package -Dmaven.test.skip=true
java -jar demo.jar --server.port=8088
```

- 配置阿里云镜像`usr/local/maven/conf/settings.xml`

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  
  <!--
  Licensed to the Apache Software Foundation (ASF) under one
  or more contributor license agreements.  See the NOTICE file
  distributed with this work for additional information
  regarding copyright ownership.  The ASF licenses this file
  to you under the Apache License, Version 2.0 (the
  "License"); you may not use this file except in compliance
  with the License.  You may obtain a copy of the License at
  
      http://www.apache.org/licenses/LICENSE-2.0
  
  Unless required by applicable law or agreed to in writing,
  software distributed under the License is distributed on an
  "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
  KIND, either express or implied.  See the License for the
  specific language governing permissions and limitations
  under the License.
  -->
  
  <!--
   | This is the configuration file for Maven. It can be specified at two levels:
   |
   |  1. User Level. This settings.xml file provides configuration for a single user,
   |                 and is normally provided in ${user.home}/.m2/settings.xml.
   |
   |                 NOTE: This location can be overridden with the CLI option:
   |
   |                 -s /path/to/user/settings.xml
   |
   |  2. Global Level. This settings.xml file provides configuration for all Maven
   |                 users on a machine (assuming they're all using the same Maven
   |                 installation). It's normally provided in
   |                 ${maven.conf}/settings.xml.
   |
   |                 NOTE: This location can be overridden with the CLI option:
   |
   |                 -gs /path/to/global/settings.xml
   |
   | The sections in this sample file are intended to give you a running start at
   | getting the most out of your Maven installation. Where appropriate, the default
   | values (values used when the setting is not specified) are provided.
   |
   |-->
  <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <!-- localRepository
     | The path to the local repository maven will use to store artifacts.
     |
     | Default: ${user.home}/.m2/repository
    <localRepository>/path/to/local/repo</localRepository>
    -->
    <localRepository>${user.home}/.m2/repository</localRepository>
    <!-- interactiveMode
     | This will determine whether maven prompts you when it needs input. If set to false,
     | maven will use a sensible default value, perhaps based on some other setting, for
     | the parameter in question.
     |
     | Default: true
    <interactiveMode>true</interactiveMode>
    -->
  
    <!-- offline
     | Determines whether maven should attempt to connect to the network when executing a build.
     | This will have an effect on artifact downloads, artifact deployment, and others.
     |
     | Default: false
    <offline>false</offline>
    -->
  
    <!-- pluginGroups
     | This is a list of additional group identifiers that will be searched when resolving plugins by their prefix, i.e.
     | when invoking a command line like "mvn prefix:goal". Maven will automatically add the group identifiers
     | "org.apache.maven.plugins" and "org.codehaus.mojo" if these are not already contained in the list.
     |-->
    <pluginGroups>
      <!-- pluginGroup
       | Specifies a further group identifier to use for plugin lookup.
      <pluginGroup>com.your.plugins</pluginGroup>
      -->
      <pluginGroup>org.mortbay.jetty</pluginGroup>
    </pluginGroups>
  
    <!-- proxies
     | This is a list of proxies which can be used on this machine to connect to the network.
     | Unless otherwise specified (by system property or command-line switch), the first proxy
     | specification in this list marked as active will be used.
     |-->
    <proxies>
      <!-- proxy
       | Specification for one proxy, to be used in connecting to the network.
       |
      <proxy>
        <id>optional</id>
        <active>true</active>
        <protocol>http</protocol>
        <username>proxyuser</username>
        <password>proxypass</password>
        <host>proxy.host.net</host>
        <port>80</port>
        <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
      </proxy>
      -->
    </proxies>
  
    <!-- servers
     | This is a list of authentication profiles, keyed by the server-id used within the system.
     | Authentication profiles can be used whenever maven must make a connection to a remote server.
     |-->
    <servers>
      <!-- server
       | Specifies the authentication information to use when connecting to a particular server, identified by
       | a unique name within the system (referred to by the 'id' attribute below).
       | 
       | NOTE: You should either specify username/password OR privateKey/passphrase, since these pairings are 
       |       used together.
       |
      <server>
        <id>deploymentRepo</id>
        <username>repouser</username>
        <password>repopwd</password>
      </server>
      -->
      
      <!-- Another sample, using keys to authenticate.
      <server>
        <id>siteServer</id>
        <privateKey>/path/to/private/key</privateKey>
        <passphrase>optional; leave empty if not used.</passphrase>
      </server>
      -->
      <server>
          <id>releases</id>
          <username>ali</username>
          <password>ali</password>
        </server>
        <server>
          <id>Snapshots</id>
          <username>ali</username>
          <password>ali</password>
        </server>
    </servers>
  
    <!-- mirrors
     | This is a list of mirrors to be used in downloading artifacts from remote repositories.
     |
     | It works like this: a POM may declare a repository to use in resolving certain artifacts.
     | However, this repository may have problems with heavy traffic at times, so people have mirrored
     | it to several places.
     |
     | That repository definition will have a unique id, so we can create a mirror reference for that
     | repository, to be used as an alternate download site. The mirror site will be the preferred
     | server for that repository.
     |-->
    <mirrors>
      <!-- mirror
       | Specifies a repository mirror site to use instead of a given repository. The repository that
       | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
       | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
       |
      <mirror>
        <id>mirrorId</id>
        <mirrorOf>repositoryId</mirrorOf>
        <name>Human Readable Name for this Mirror.</name>
        <url>http://my.repository.com/repo/path</url>
      </mirror>
       -->
      <mirror>
        <!--This sends everything else to /public -->
        <id>nexus</id>
        <mirrorOf>*</mirrorOf> 
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      </mirror>
      <mirror>
        <!--This is used to direct the public snapshots repo in the 
            profile below over to a different nexus group -->
        <id>nexus-public-snapshots</id>
        <mirrorOf>public-snapshots</mirrorOf> 
        <url>http://maven.aliyun.com/nexus/content/repositories/snapshots/</url>
      </mirror>
      <mirror>
        <!--This is used to direct the public snapshots repo in the 
            profile below over to a different nexus group -->
        <id>nexus-public-snapshots1</id>
        <mirrorOf>public-snapshots1</mirrorOf> 
        <url>https://artifacts.alfresco.com/nexus/content/repositories/public/</url>
      </mirror>
    </mirrors>
  
    <!-- profiles
     | This is a list of profiles which can be activated in a variety of ways, and which can modify
     | the build process. Profiles provided in the settings.xml are intended to provide local machine-
     | specific paths and repository locations which allow the build to work in the local environment.
     |
     | For example, if you have an integration testing plugin - like cactus - that needs to know where
     | your Tomcat instance is installed, you can provide a variable here such that the variable is
     | dereferenced during the build process to configure the cactus plugin.
     |
     | As noted above, profiles can be activated in a variety of ways. One way - the activeProfiles
     | section of this document (settings.xml) - will be discussed later. Another way essentially
     | relies on the detection of a system property, either matching a particular value for the property,
     | or merely testing its existence. Profiles can also be activated by JDK version prefix, where a
     | value of '1.4' might activate a profile when the build is executed on a JDK version of '1.4.2_07'.
     | Finally, the list of active profiles can be specified directly from the command line.
     |
     | NOTE: For profiles defined in the settings.xml, you are restricted to specifying only artifact
     |       repositories, plugin repositories, and free-form properties to be used as configuration
     |       variables for plugins in the POM.
     |
     |-->
     <profiles> 
      <profile>
        <id>development</id>
        <repositories>
          <repository>
            <id>central</id>
            <url>http://central</url>
            <releases><enabled>true</enabled><updatePolicy>always</updatePolicy></releases>
            <snapshots><enabled>true</enabled><updatePolicy>always</updatePolicy></snapshots>
          </repository>
        </repositories>
       <pluginRepositories>
          <pluginRepository>
            <id>central</id>
            <url>http://central</url>
            <releases><enabled>true</enabled><updatePolicy>always</updatePolicy></releases>
            <snapshots><enabled>true</enabled><updatePolicy>always</updatePolicy></snapshots>
          </pluginRepository>
        </pluginRepositories>
      </profile>
      <profile>
        <!--this profile will allow snapshots to be searched when activated-->
        <id>public-snapshots</id>
        <repositories>
          <repository>
            <id>public-snapshots</id>
            <url>http://public-snapshots</url>
            <releases><enabled>false</enabled></releases>
            <snapshots><enabled>true</enabled><updatePolicy>always</updatePolicy></snapshots>
          </repository>
        </repositories>
       <pluginRepositories>
          <pluginRepository>
            <id>public-snapshots</id>
            <url>http://public-snapshots</url>
            <releases><enabled>false</enabled></releases>
            <snapshots><enabled>true</enabled><updatePolicy>always</updatePolicy></snapshots>
          </pluginRepository>
        </pluginRepositories>
      </profile>
    </profiles>
   
     <activeProfiles>
      <activeProfile>development</activeProfile>
      <activeProfile>public-snapshots</activeProfile>
     </activeProfiles>
  
    <!-- activeProfiles
     | List of profiles that are active for all builds.
     |
    <activeProfiles>
      <activeProfile>alwaysActiveProfile</activeProfile>
      <activeProfile>anotherAlwaysActiveProfile</activeProfile>
    </activeProfiles>
    -->
  </settings>
  ```

## 2.3、安装git

```
yum install git
```

## 2.4、下载Jenkins&启动

```shell
#下载jenkins.war：https://get.jenkins.io/war-stable/2.319.3/
java -jar jenkins.war
#开启防火墙
service firewalld start
#添加http服务到firewalld,pemmanent表示永久生效，若不加–permanent系统下次启动后就会失效
sudo firewall-cmd --permanent --add-service=http
#重启防火墙
sudo systemctl reload firewalld
#查看端口号是否已开
firewall-cmd --query-port=8080/tcp
#添加指定需要开放的端口
firewall-cmd --add-port=8080/tcp --permanent
#重载入添加的端口
firewall-cmd --reload
#查询指定端口是否开启成功
firewall-cmd --query-port=8080/tcp
```

![image-20221123220832111](./assets/image-20221123220832111.png)

![image-20221123221042592](./assets/image-20221123221042592.png)

- 首次启动war包会在`/root/.jenkins`生成配置文件；


- 密码文件使用后会自动删除；


## 2.5、Jenkins + Git + Maven 自动化部署配置

### 2.5.1、安装maven插件

![image-20221123232140447](./assets/image-20221123232140447.png)

![image-20221123232319481](./assets/image-20221123232319481.png)

![image-20221123232443440](./assets/image-20221123232443440.png)

### 2.5.2、Git配置

![image-20221123234126578](./assets/image-20221123234126578.png)

![image-20221123234011905](./assets/image-20221123234011905.png)

![image-20221123234259105](./assets/image-20221123234259105.png)

![image-20221123234411948](./assets/image-20221123234411948.png)

### 2.5.3、Maven配置

![image-20221123234846416](./assets/image-20221123234846416.png)

![image-20221123235048742](./assets/image-20221123235048742.png)

### 2.5.4、pom.xml配置

![image-20221123235617141](./assets/image-20221123235617141.png)

![image-20221126164904831](./assets/image-20221126164904831.png)

### 2.5.5、配置测试服务器

1. 创建测试服务器，安装JDK，开启对应的端口；

2. 添加`Publish Over SSH`插件：

   ![image-20221126175940653](./assets/image-20221126175940653.png)

3. 在Jenkins的全局配置中添加`Publish Over SSH`的配置（添加一台目标服务器）：

   ![image-20221126180635229](./assets/image-20221126180635229.png)

   ![image-20221126181110548](./assets/image-20221126181110548.png)

   ![image-20221126181156119](./assets/image-20221126181156119.png)

4. 回到构建项目配置中，完善`Post Steps`配置：

   ![image-20221126181546628](./assets/image-20221126181546628.png)

![image-20221126183028388](./assets/image-20221126183028388.png)

![image-20221126183149753](./assets/image-20221126183149753.png)

- 推送到测试服务器后，可在服务器上通过`jps`查看（显示当前所有java进程pid的命令）；

![image-20221126185533321](./assets/image-20221126185533321.png)

![image-20221126190137901](./assets/image-20221126190137901.png)

### 2.5.6、超时机制

- 在构建完毕后执行的命令如果花费的时间比较长，或者是执行会让窗口卡住的命令，那么Jenkins就会认为一直还没有执行完，就会报超时错误；

  ![image-20221126191429968](./assets/image-20221126191429968.png)

![image-20221126185735692](./assets/image-20221126185735692.png)

- 解决方式：可让该命令的提示日志在后台输出到指定文件中；

  ```
  nohup java -jar /root/xxoo/demo*.jar >mylog.log 2>&1 &
  ```

  ![image-20221126192036564](./assets/image-20221126192036564.png)

### 2.5.7、数据流重定向

- 数据流重定向就是将某个命令执行后，应该要出现在屏幕上的数据传输到其他地方：
  - 标准输入（stdin）：代码为0，使用<或<<；
  - 标准输出（stdout）：代码为1，使用>或>>；
  - 标准错误输出（stderr）：代码为2，使用2>或2>>（>覆盖写、\>> 追加写）；

### 2.5.8、运行前清理

- 执行前赋权限`chomd 777 pre-steps.sh`

```shell
#!/bin/bash

#删除历史数据
rm -rf xxoo

appname=$1
#获取传入的参数
echo "arg:$1"

#获取正在运行的jar包pid
pid=`ps -ef | grep $1 | grep 'java -jar' | awk '{printf $2}'`

echo $pid

#如果pid为空，提示一下，否则，执行kill命令
if [ -z $pid ];
#使用-z 做空值判断
        then
                echo "$appname not started"
        else
                kill -9 $pid
                echo "$appname stoping...."

check=`ps -ef | grep -w $pid | grep java`
if [ -z $check ];
        then
                echo "$appname pid:$pid is stop"
        else
                echo "$appname stop failed"

fi

fi
```

![image-20221126204752346](./assets/image-20221126204752346.png)

![image-20221126205208000](./assets/image-20221126205208000.png)

### 2.5.9、几种构建方式

#### 2.5.9.1、触发远程构建

- 安装插件：https://plugins.jenkins.io/build-token-root/
- Jenkins中设置token，提供rest api；
- 在GitLab的提交等事件中，携带指定token，触发调用Jenkins提供的api，即一旦GitLab上的代码发生变化会通知Jenkins进行构建；

![image-20221127103118367](./assets/image-20221127103118367.png)

![image-20221127104708629](./assets/image-20221127104708629.png)

![image-20221127105146756](./assets/image-20221127105146756.png)

![image-20221127104859346](./assets/image-20221127104859346.png)

#### 2.5.9.2、快照依赖构建

- Build whenever a SNAPSHOT dependency is built；

- 当依赖的快照被构建时执行本job；

  ![image-20221127110002819](./assets/image-20221127110002819.png)

#### 2.5.9.3、工程依赖构建

- Build after other projects are built；
- 适用于父子工程，当子工程构建完成之后再执行本job；

#### 2.5.9.4、定时构建

- Build periodically；

- 使用cron表达式定时构建本job；

  ![image-20221127110459717](./assets/image-20221127110459717.png)

  ##### 2.5.9.4.1、Jenkins cron表达式

  - 标准cron：https://crontab.guru


  - Jenkins cron不是标准的cron表达式；

    ```
    第一个 * 表示每个小时的第几分钟，取值0~59
    H * * * *
    H：每小时执行一次
    
    第二颗 * 表示小时，取值0~23
    * 15 * * * 表示每天下午3点
    * 1 * * *  表示每天凌晨1点
    
    第三颗 * 表示一个月的第几天，取值1~31
    * 1 5 * *  表示每月5日凌晨1点
    
    第四颗 * 表示第几月，取值1~12
    * 15 5 1 *  表示每年几月执行
    
    第五颗 * 表示一周中的第几天，取值0~7，其中0和7代表的都是周日
    ```

  - **“/”**：表示每隔多长时间，比如 */10 * * * * 表示每隔10分钟；

  - **“H”**：hash散列值，以job名取值，获取到以job名为入参的唯一值，相同名称值也相同，这个偏移量会和实际时间相加，获得一个真实的运行时间。意义在于：不同的项目在不同的时间运行，即使配置的值是一样的，比如，都是`15 * * * * ` ，表示每个小时的第15分钟开始执行任务，那么会造成同一时间内在Jenkins中启动很多job，换成`H/15 * * * *`,那么在首次启动任务时，会有随机值参与进来，有的会在17分钟启动有的会在19分钟启动，随后的启动时间也是这个值。这样就能错开相同cron值的任务执行了；

  - H的值也可以设置范围：
    - `H * * * *`表示一小时内的任意时间；
    - `*/10 * * * *`每10分钟；
    - `H/10 * * * *`每10分钟，可能是7，17，27，起始时间hash，步长不变；
    - `45 3 * * 1-6 `每个周一至周六，凌晨3点45执行1次；
    - `45 3-5 * * 1-6 `每个周一至周六，凌晨3点45 ，凌晨4点45，凌晨5点45各执行1次；
    - `H(40-48) 3-5 * * 1-6 `在40~48之间取值，其他同上；
    - `45 3-5/2 * * 1-6 `每个周一至周六，凌晨3点45 ，凌晨5点45各执行1次；
    - ` 45 0-6/2 * * 1-6 * * 1-6 `0点开始，每间隔2小时执行一次，0:45、2:45、4:45；

#### 2.5.9.5、GitHub提交代码时触发Jenkins自动构建

- GitHub hook trigger for GITScm polling；

- Github-WebHook触发时构建本job，但是一般项目的代码是不会托管Github平台上；

  ![image-20221127110728971](./assets/image-20221127110728971.png)

#### 2.5.9.6、定期检查代码变更

- Poll SCM；

- 由Jenkins自动发起的，使用cron表达式定时检查代码变更，变更后构建本job；

- 使用Poll SCM 方式与Build periodically一样，会主动定期检查代码托管服务器上是否有变化，一旦发生变化执行job构建；

  ![image-20221127111036052](./assets/image-20221127111036052.png)

### 2.5.10、测试报告邮件通知

无

# 3、自动化部署到Docker容器中

## 3.1、打包到容器内

- 创建服务器，安装Docker；

- 配置国内镜像：

  - 修改`/etc/docker/daemon.json`，不存在的话则创建；

    ```json
    {
        "registry-mirrors": [
            "https://ustc-edu-cn.mirror.aliyuncs.com",
            "http://hub-mirror.c.163.com",
            "https://registry.aliyuncs.com"
        ]
    }
    ```

  - 重启服务：

    ```shell
    systemctl daemon-reload
    systemctl restart docker
    ```

- 官网下载镜像：https://hub.docker.com/

![image-20221127123022558](./assets/image-20221127123022558.png)

- 参照官网命令：

  ```dockerfile
  FROM openjdk:11
  COPY . /usr/src/myapp
  WORKDIR /usr/src/myapp
  RUN javac Main.java
  CMD ["java", "Main"]
  ```

  `vi dockerfile`

  ```dockerfile
  #指定镜像
  FROM openjdk:11
  #开放端口
  EXPOSE 8080
  #工作目录，上下文
  WORKDIR /root
  #添加jar包
  ADD jarfile/demo*.jar /root/app.jar
  #执行命令生成
  ENTRYPOINT ["java","-jar","/root/app.jar"]
  ```

- 打包镜像demo：`docker build -t demo .`（.代表当前目录）；

- 查看镜像`docker images`，openjdk是拉取的镜像，demo是刚刚打包的镜像；

  ![image-20221127125205283](./assets/查看镜像.png)

- 开放端口：https://blog.csdn.net/weixin_51687288/article/details/121415885

  ```shell
  #开启防火墙
  service firewalld start
  #添加http服务到firewalld,pemmanent表示永久生效，若不加–permanent系统下次启动后就会失效
  sudo firewall-cmd --permanent --add-service=http
  #重启防火墙
  sudo systemctl reload firewalld
  #查看端口号是否已开
  firewall-cmd --query-port=8099/tcp
  #添加指定需要开放的端口
  firewall-cmd --add-port=8099/tcp --permanent
  #重载入添加的端口
  firewall-cmd --reload
  #查询指定端口是否开启成功
  firewall-cmd --query-port=8099/tcp
  ```

- 运行&查看：

  ```shell
  #运行
  docker run -d --name demo -p 8080:8080 demo
  #查看正在运行的容器
  docker ps
  ```

## 3.2、Docker外挂目录

```shell
#查看最新创建的容器，包含所有状态
docker ps -l
docker ps -a
#拿到容器ID，删除对应的容器
docker rm 5029f0dd6e8f
#根据CONTAINER ID关闭某个正在运行的容器
docker stop 9caf5777264d
#通过JDK影像，配置外挂目录的方式启动容器
docker run -d -p 8080:8080 --name demo-out -v /root/jarfile/demo-0.0.1-SNAPSHOT.jar:/app.jar openjdk:11 java -jar app.jar
```

## 3.3、Jenkins+Docker外挂目录构建

- 更改测试服务器地址：

  ![image-20221127134454914](./assets/image-20221127134454914.png)

- 构建前删除jar包，停止容器：

  ![image-20221127140113148](./assets/image-20221127140113148.png)

- 构建后发送jar包，运行容器：

  ![image-20221127140239662](./assets/image-20221127140239662.png)

## 3.4、Jenkins+Docker重新打包

- 在IDEA中创建dockerfile；

  ![image-20221127144314532](./assets/image-20221127144314532.png)

- 构建前：

  - 删除jar包；

  - 删除dockerfile；

  - 停止容器，删除容器；

  - 删除镜像文件；

    ![image-20221127144137448](./assets/image-20221127144137448.png)

- 构建后：

  - 发送jar包；

  - 通过命令执行dockerfile生成镜像；

  - 通过执行镜像生成容器；

    ![image-20221127144228770](./assets/image-20221127144228770.png)

# 4、Jenkins集群/并发构建

![image-20221127154636209](./assets/image-20221127154636209.png)

![image-20221127154749941](./assets/image-20221127154749941.png)

![image-20221127155211598](./assets/image-20221127155211598.png)

![image-20221127155321669](./assets/image-20221127155321669.png)

![image-20221127155531088](./assets/image-20221127155531088.png)

![image-20221127155819254](./assets/image-20221127155819254.png)

![image-20221127160429305](./assets/image-20221127160429305.png)

# 5、流水线pipeline

## 5.1、介绍&语法

- 流水线既能作为任务的本身，也能作为Jenkinsfile；

- 使用流水线可以让我们的任务从ui手动操作，转换为代码化，像docker的dockerfile一样，从shell命令到配置文件，更适合大型项目，可以让团队其他开发者同时参与进来，同时也可以编辑开发Jenkinswebui不能完成的更复杂的构建逻辑，作为开发者可读性也更好；

- 完整语法：

  - 5个必备的组成部分：

    ```
    pipeline：整条流水线
    agent：指定执行器
    stages：所有阶段
    stage：某一阶段，可有多个
    steps：阶段内的每一步，可执行命令
    ```

  - post：流水线完成后可执行的任务：

    ```
    always：无论流水线或者阶段的完成状态。
    changed：只有当流水线或者阶段完成状态与之前不同时。
    failure：只有当流水线或者阶段状态为"failure"运行。
    success：只有当流水线或者阶段状态为"success"运行。
    unstable：只有当流水线或者阶段状态为"unstable"运行。例如：测试失败。
    aborted：只有当流水线或者阶段状态为"aborted "运行。例如：手动取消。
    ```

  - agent：可以指定执行节点

    ```
    label：指定运行job的节点标签
    any：不指定，由Jenkins分配
    ```

- 示例：

  ```json
  pipeline {
      agent {
          node {
              label "jenkins-02"
          }
      }
      stages {
          stage('拉取代码') {
              steps {
                  sh """
                      sleep 10       
                     """
                  echo '拉取代码完成'
              }
          }
          stage('执行构建') {
              steps {
                  echo '执行构建完成'
              }
          }
      }
      post {
          always {
              echo "完成"
          }
          failure {
              echo "失败"
          }
      }
  }
  ```

  ![image-20221127161527591](./assets/image-20221127161527591.png)

  ![image-20221127161359477](./assets/image-20221127161359477.png)

![image-20221127162019176](./assets/image-20221127162019176.png)

![image-20221127163353813](./assets/image-20221127163353813.png)

## 5.2、Blue Ocean

- 可视化界面，全新的流水线控制ui，可重复执行某阶段代码；
- 插件中心搜索blue ocean安装即可;

![image-20221127165513213](./assets/image-20221127165513213.png)

![image-20221127165124253](./assets/image-20221127165124253.png)

![image-20221127171626760](./assets/image-20221127171626760.png)

![image-20221127173826602](./assets/image-20221127173826602.png)

![image-20221127174017170](./assets/image-20221127174017170.png)

```json
pipeline {
    agent any
    tools {
        maven "maven3"
    }
    stages {
        stage('拉取代码') {
            steps {
                git branch: 'main', credentialsId: 'gitlab', url: 'http://192.168.182.102/root/java-project.git'
                echo '拉取成功'
            }
        }
        stage('开始构建') {
            steps {
                //sh "mvn -version"
                sh """
                mvn -version
                pwd
                mvn clean package
                """
                echo '构建成功'
            }
        }
        stage('开始清理') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'test-server', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''rm -rf  jarfile/
rm -rf dockfile
docker stop demo
docker rm demo
docker rmi demo''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/root', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
                echo '清理成功'
            }
        }
        stage('开始创建容器') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'test-server', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'docker/dockerfile'), sshTransfer(cleanRemote: false, excludes: '', execCommand: '''docker build -t demo .
docker run -d --name demo -p 8080:8080 demo''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/jarfile', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: '**/demo*.jar')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
                echo '创建容器成功'
            }
        }
    }
}
```

# 6、使用Jenkins file创建多分支Job

![image-20221127180141833](./assets/image-20221127180141833.png)

![image-20221127180450918](./assets/image-20221127180450918.png)

![image-20221127181542712](./assets/image-20221127181542712.png)

![image-20221127181626657](./assets/image-20221127181626657.png)

![image-20221127181853853](./assets/image-20221127181853853.png)

![image-20221127181700113](./assets/image-20221127181700113.png)

![image-20221127181754540](./assets/image-20221127181754540.png)

# 7、声明式和脚本式流水线比较

## 7.1、声明式流水线

- 好处：
  - 更像是在Jenkins web ui中的操作；
  - 可读性比较高；
  - 可以使用blue ocean自动生成；
  - 支持语法检查；

- 坏处：
  - 代码逻辑能力比脚本式弱，不能完成特别复杂的任务；

## 7.2、脚本式流水线

- 好处：
  - 更少的代码和弱规范要求；
  - 更灵活的自定义代码操作；
  - 不受约束，可以构建特别复杂的工作流和流水线；

- 坏处：
  - 读写对编程要求比较高；
  - 比声明式流水线代码更复杂；



