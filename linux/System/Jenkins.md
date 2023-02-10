## 1.CI/CD

`1.持续集成 CI`

开发人员提交代码，整合代码的过程



`2.持续交付 CD (DELIVERY)`

运维人员拉取代码，部署到生产环境；持续交付就是讲我们的应用发布出去的过程

如；苹果发布ios 14



`3.持续部署 CD (DEPLOYMENT)`

部署，我们点击手机更新 ios 14

## 2.安装

centos系统：

```shell
$ sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
$ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
$ sudo yum upgrade
# Add required dependencies for the jenkins package
$ sudo yum install java-11-openjdk
$ sudo yum install jenkins
$ sudo systemctl daemon-reload
```

访问：http://ip:8080

---



## 3.配置

依赖环境安装
因为需要对一些项目进行打包，因此需要安装这些环境的依赖，这里我们就安装jdk、git、maven(java项目)，nodejs(前端项目)即可。

### 1.maven安装配置

```shell
$ cd /usr/local
$ wget --no-check-certificate https://dlcdn.apache.org/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
$ tar -xf apache-maven-3.6.3-bin.tar.gz
$ mv apache-maven-3.6.3 maven
$ vim /etc/profile     # 配置环境变量
MAVEN_HOME=/usr/local/maven
export PATH=${MAVEN_HOME}/bin:${PATH}

$ source /etc/profile
$ mvn -v
# 打包
$ mvn clean package -Dmaven.test.skip=true
```

配置指定jdk版本信息：  以jdk11为例

- 配置Maven环境中的JDK版本，该方法适用于该环境下的所有项目，一劳永逸

```xml
$ vim /usr/local/maven/conf/settings.xml

    <profile>
      <id>jdk-11</id>
      <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>11</jdk>
      </activation>
      <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <maven.compiler.compilerVersion>11</maven.compiler.compilerVersion>
      </properties>
    </profile>
```

jenkins配置maven环境

- 系统管理---系统配置---全局属性

![image-20220906185138617](D:\Tech\linux\System\.assets\image-20220906185138617.png)

### 2.nodejs安装配置

node下载地址：
http://nodejs.cn/download/current/
https://nodejs.org/dist/

```
tar -zxvf node-v11.15.0-linux-x64.tar.gz
mv node-v11.15.0-linux-arm64/ nodejs
mv nodejs/* /usr/local/nodejs
ln -s /usr/local/nodejs/bin/node /usr/local/bin
ln -s /usr/local/nodejs/bin/npm /usr/local/bin
npm install -g yarn
ln -s /usr/local/nodejs/bin/yarn /usr/local/bin

$ yarn -v
$ npm -v
$ node -v

```

### 3.Git配置

Jenkins - SSH认证方式拉取Git代码

```shell
$ ssh-keygen   # jenkins服务器生成密钥对，并将公钥上传git
```

![image-20220906221430864](D:\Tech\linux\System\.assets\image-20220906221430864.png)

新增任务时，添加git一直报错，想到是jenkins默认启动用户是jenkins，而我们上传到git的公钥是root用户的，在此我修改jenkins的启动用户为root：

```shell
$ vim /etc/sysconfig/jenkins
JENKINS_USER="root"

$ vim /usr/lib/systemd/system/jenkins.service
User=root
Group=root

$ systemctl daemon-reload
$ systemctl restart jenkins
```

### 4.相关插件安装

a. 安装配置 Publish Over SSH 插件

![image-20220907094709545](D:\Tech\linux\System\.assets\image-20220907094709545.png)

配置ssh远程主机：

- 系统管理---系统配置---Publish over SSH

  

  

## 4.参数化构建

`i.字符参数(String Parameter)`

![image-20230105102354156](D:\Tech\linux\System\.assets\image-20230105102354156.png)

`ii.选项参数(Choice Parameter)`

![image-20230105102524965](D:\Tech\linux\System\.assets\image-20230105102524965.png)

`iii.构建`

![image-20230105102654510](D:\Tech\linux\System\.assets\image-20230105102654510.png)

---



## 5.管理员密码忘记

1.删除Jenkins目录下config.xml文件中下面代码并保存

![image-20230104180026396](D:\Tech\linux\System\.assets\image-20230104180026396.png)

2.重启jenkins

```shell
$ systemctl restart jenkins
```

3.进入首页>“系统管理”>“Configure Global Security”；点选“Jenkins专有用户数据库”，并点击“保存”

![image-20230104181311817](D:\Tech\linux\System\.assets\image-20230104181311817.png)

4.重新点击首页>“系统管理”,发现此时出现“管理用户”

![image-20230104181419505](D:\Tech\linux\System\.assets\image-20230104181419505.png)

然后修改密码！！

5.禁止匿名用户登陆

![image-20230104185746519](D:\Tech\linux\System\.assets\image-20230104185746519.png)

