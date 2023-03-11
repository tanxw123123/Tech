## 1.JDK安装



**1.Oracle JDK**

[下载jdk11](https://www.oracle.com/java/technologies/downloads/#java11)

[下载jdk8](https://www.oracle.com/java/technologies/javase/javase8u211-later-archive-downloads.html#license-lightbox)

```shell
- 安装
$ sudo mkdir  -p /opt/jdk
# 解压后放到/opt/jdk

$ sudo update-alternatives --install /usr/bin/java java /opt/jdk/jdk1.8.0_251/bin/java 100
```

配置环境变量：

```shell
# 编辑 /etc/environment ，填入以下内容
JAVA_HOME=/opt/jdk/jdk1.8.0_251
JRE_HOME=/opt/jdk/jdk1.8.0_251/jre
# 使生效
source /etc/environment
```



**2.OpenJDK**

```shell
$ apt install openjdk-8-jdk
$ apt install openjdk-11-jdk
```

## 2.JDK卸载

```shell
$ dpkg --list | grep -i jdk            # 先检查是否安装
$ apt-get purge openjdk*               # 移除openjdk包
$ apt-get purge icedtea-* openjdk-*    # 卸载 OpenJDK 相关包
$ dpkg --list | grep -i jdk            # 再次检查是否卸载成功

- 和 remove 不同的是，remove 只是删掉数据和可执行文件，purge 另外还删除所有的配制文件。
```

