## 1.阿里云API

1.阿里云cli客户端

阿里云命令行工具（Alibaba Cloud Command Line Interface）是在Alibaba Cloud SDK for Go之上构建的开源工具。您可以在命令行Shell中，使用aliyun命令与阿里云服务进行交互，管理您的阿里云资源



安装：

```shell
$ wget https://aliyuncli.alicdn.com/aliyun-cli-linux-latest-amd64.tgz?spm=a2c4g.11186623.0.0.1e525d8cdvgi2s&file=aliyun-cli-linux-latest-amd64.tgz

$ tar -xf aliyun-cli-linux-latest-amd64.tgz
$ cp aliyun /usr/bin/

在GitHub上下载您所需版本的阿里云CLI
https://github.com/aliyun/aliyun-cli/releases?spm=a2c4g.11186623.0.0.1e525d8cdvgi2s
```

配置：

```shell
# 交互式配置：
$ aliyun configure --profile aktest

--profile: 指定配置名称。如果指定的配置存在，则修改配置。若不存在，则创建配置。

$ aliyun configure list     # 查看
$ aliyun configure delete--profile aktest
```

```shell
# 非交互式配置
$ aliyun configure set --profile aktest --region cn-hangzhou --access-key-id xxxxx --access-key-secret xxxxxxxx
```

```shell
$ aliyun --help           # 查看帮助
```



## 2.腾讯云API

**我们推荐您使用 [腾讯云 API Explorer](https://console.cloud.tencent.com/api/explorer) ，[腾讯云 SDK](https://cloud.tencent.com/document/sdk) 和 [腾讯云命令行工具（TCCLI）](https://cloud.tencent.com/product/cli) 等开发者工具，从而无需学习如何对 API 请求进行签名。**



1.命令行工具 TCCLI

https://cloud.tencent.com/document/product/440/34011

安装

```shell
$ git clone https://github.com/TencentCloud/tencentcloud-cli.git
$ cd tencentcloud-cli
$ python setup.py install
```

配置

```shell
$ tccli configure
```

```shell
$ tccli help                # 查看帮助
tccli help --detail         # 查看详细帮助
$ tccli billing help    # 余额查询帮助
```

```shell
# 查询余额

$ tccli billing DescribeAccountBalance

报错：you are not authorized to perform operation (finance:DescribeAccountBalance)

搜索： finance
授权： QCloudFinanceFullAccess

- 再次访问正常！
```

```shell
# 查询直播流

$ tccli live help
$ tccli live DescribeLiveStreamOnlineList

授权： QcloudLIVEReadOnlyAccess
```

## 3.7牛云

qshell安装配置

[下载安装包](https://devtools.qiniu.com/qshell-v2.10.0-linux-386.tar.gz?ref=developer.qiniu.com&s_path=%2Fkodo%2F1302%2Fqshell)

```shell
Usage:
  qshell account [<AccessKey> <SecretKey> <UserName>] [flags]
  
$ qshell account ak sk name
```

## 4.优刻得API

https://console.ucloud.cn/uapi/ucloudapi/Computing

```shell
$ git clone https://github.com/ucloud/ucloud-cli.git
$ cd ucloud-cli/
$ apt install golang
$ make install

$ ucloud init  初始化，写入公钥和私钥
$ ucloud api   --Action GetBalance    # 余额查询接口
```

余额查询及告警脚本：

```shell
#!/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
balance=`ucloud api   --Action GetBalance | awk '/AmountAvailable/{print}'|awk -F'\"' '{print $4}'|awk '{print int($0)}'`
if [ $balance -le 4000 ];then
    message="""==============优刻得云账号==============
当前余额: $balance元; 请及时充值"""
    curl -X POST "https://api.telegram.org/bot5429948026:AAFtWaW1ZcU4IUNaNNtIFfK9qHIYoMjAlgg/sendMessage" -d "chat_id=-613903645&text=$message" &> /dev/null
fi
```

```shell
# 上面 PATH变量必须加，否则计划任务找不到ucloud命令，可以通过日志查看
$ tail -f /var/spool/mail/root
$ tail -f /var/log/cron.log
```

