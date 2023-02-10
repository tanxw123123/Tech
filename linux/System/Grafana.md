# Grafana

## 1.安装

centos系统：

```shell
$ curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
$ yum clean all
$ yum makecache
$ yum repolist
$ yum update
$ vim /etc/yum.repos.d/grafana.repo
[grafana]
name=grafana
baseurl=https://mirrors.aliyun.com/grafana/yum/rpm
repo_gpgcheck=0
enabled=1
gpgcheck=0

$ yum makecache
$ yum repolist
$ yum -y install grafana
$ systemctl start grafana-server
$ systemctl enable grafana-server
$ ss -tnlp|grep 3000
```

ubuntu系统：

```shell
$ sudo apt update
$ sudo apt upgrade
$ sudo apt-get install -y gnupg2 curl software-properties-common
$ curl https://packages.grafana.com/gpg.key | sudo apt-key add -
$ sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
$ sudo apt update
$ sudo apt install grafana
$ sudo systemctl start grafana-server
$ sudo systemctl enable grafana-server
```

打开浏览器，输入http://ip:3000(端口为3000)，打开Grafana控制面板， 初始默认账号和密码均为 admin，初次登录需要修改密码。



docker安装：

```shell
$ docker run -d -p 3000:3000 --name grafana grafana/grafana-enterprise:8.2.0
$ mkdir /data/grafana/data

$ docker cp grafana:/var/lib/grafana /data/grafana/data
$ docker cp grafana:/etc/grafana/grafana.ini /data/grafana/grafana.ini
$ chmod -R 777 /data/grafana

$ docker run -d -p 3000:3000 -v /data/grafana/data:/var/lib/grafana -v /data/grafana/grafana.ini:/etc/grafana/grafana.ini --name grafana grafana/grafana-enterprise:8.2.0
```

---



## 2.配置数据源

点击设置--选择Data sources：

![image-20220905210859915](D:\Tech\linux\System\.assets\image-20220905210859915.png)

然后将prometheus server地址填写：

![image-20220905211132545](D:\Tech\linux\System\.assets\image-20220905211132545.png)

---



## 3.导入监控模板

https://grafana.com/grafana/dashboards

比如，我这里导入证书监控模板，搜索：ssl

我这里选择导入第一个：

![image-20220905211655901](D:\Tech\linux\System\.assets\image-20220905211655901.png)

拷贝模板ID：

![image-20220905211755793](D:\Tech\linux\System\.assets\image-20220905211755793.png)

导入模板：

![image-20220905211915710](D:\Tech\linux\System\.assets\image-20220905211915710.png)

将ID粘贴过来：

![image-20220905211951512](D:\Tech\linux\System\.assets\image-20220905211951512.png)

选择数据源，然后导入：

![image-20220905212105034](D:\Tech\linux\System\.assets\image-20220905212105034.png)



