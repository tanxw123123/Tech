# CentOS7使用NTP搭建时间同步服务器

为什么要搭建时间同步[服务器](https://cloud.tencent.com/product/cvm?from=10680)呢？场景是这样的。

我们有两台CentOS服务器，其中一台是可以连接外网的，下文中我们称它为A服务器，另一台不能连接外网，但是与A服务器在同一局域网中，下文中我们称它为服务器B。

现在我们需要将A服务器的时间进行网络校准，这部分操作还是比较容易的，按照下面的步骤操作即可。

1、yum进行ntp的安装：

```shell
$ yum -y install ntp
```

2、执行同步命令：

```shell
$ ntpdate time1.aliyun.com
```

Centos7默认通过chronyd服务实现时钟同步，我们需要关闭chronyd服务并使其开机不自启，同时启动ntpd并将其加入开机自启：

```shell
$ systemctl stop chronyd
$ systemctl disable chronyd 
$ systemctl enable ntpd
$ systemctl start ntpd
```

接下来就是去修改ntp的配置文件了：

```shell
$ vim /etc/ntp.conf

#1 把下边这行注释掉
# restrict default nomodify notrap nopeer noquery
#2 删除掉原有的4行server，增加下边的两行，127.127.1.0代表把本机作为时间服务器
server 127.127.1.0
fudge   127.127.1.0 stratum 10
```

修改后重新启动NTP服务即可。

```shell
$ systemctl restart ntpd
```

到这里其实我们的时间服务器就搭建完成了，现在我们只要在B服务器上执行下边的命令就可以进行时间同步了。

```shell
$ ntpdate A服务器的IP地址
```

实际的情况，我们不应该去手动执行时间同步命令，应该设置一个定时任务，每隔多长时间就自动去进行一次时间校对工作。

我们可以直接执行如下命令：

```shell
$ crontab -e
0 */1 * * * ntpdate 192.160.99.201
```

