服务环境

- 系统：ubuntu20.04

- 主机名：zulip-server

- ip: 46.51.254.x

- 域名：zulip.oball50.com

## 1.安装openssl

```shell
$ apt install openssl
```

## 2.下载&安装

```shell
# 下载
$ wget -P /data https://www.zulip.org/dist/releases/zulip-server-latest.tar.gz
$ cd /data && tar -xf zulip-server-latest.tar.gz
$ sudo -s ./zulip-server-*/scripts/setup/install --certbot --email=你的email --hostname=zulip.oball50.com
...
https://zulip.oball50.com/new/uoh2qqudsjkqaoa2azft7n6

#安装命令中可以带两种参数：
  --cerbot：用于线上product环境搭建，需要有域名SSL证书；
  --self-signed-cert：用于测试服务器搭建，Zulip安装程序会生成只签名SSL证书，需要确保安装OpenSSL。
```

## 3.邮箱配置（google邮箱为例）

### 1.开启邮箱IMAP

登录邮箱-设置-查看所有设置：

![image-20221018111752119](D:\Tech\linux\Ronnie\.assets\image-20221018111752119.png)

点击转发和POP/IMAP，启用IMAP，默认为停用状态，保存配置

![image-20221018111931241](D:\Tech\linux\Ronnie\.assets\image-20221018111931241.png)

获取授权码：

点击google头像--管理你的google账号--安全性，开启两步验证

然后，选择应用专用密码，即为授权码

![image-20221018115438041](D:\Tech\linux\Ronnie\.assets\image-20221018115438041.png)

**b.修改配置文件**

```shell
$ vim /etc/zulip/settings.py
EMAIL_HOST = "smtp.gmail.com"
EMAIL_HOST_USER = "xxxx@gmail.com"
EMAIL_USE_TLS = True
EMAIL_PORT = 587
ADD_TOKENS_TO_NOREPLY_ADDRESS = False
NOREPLY_EMAIL_ADDRESS = "xxxx@gmail.com"

$ vim /etc/zulip/zulip-secrets.conf
# 末尾添加：
email_password = aozofiabcdbqrmxn     # 此处为邮箱的授权码而不是密码
```

重启：

```shell
# 重启
$ su zulip -c'/home/zulip/deployments/current/scripts/restart-server'
# 发邮箱测试

```

发邮箱测试：

```shell
$ su zulip -c '/home/zulip/deployments/current/manage.py send_test_email xxxx@gmail.com'
...
Sending 2 test emails from:
  * xxxx@gmail.com
  * xxxx@gmail.com

Successfully sent 2 emails to xxxx@gmail.com!
```

