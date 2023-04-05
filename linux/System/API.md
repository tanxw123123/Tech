## 1.阿里云API

### 1.阿里云cli客户端

阿里云命令行工具（Alibaba Cloud Command Line Interface）是在Alibaba Cloud SDK for Go之上构建的开源工具。您可以在命令行Shell中，使用aliyun命令与阿里云服务进行交互，管理您的阿里云资源



安装：

[包下载地址](https://aliyuncli.alicdn.com/aliyun-cli-linux-latest-amd64.tgz?spm=a2c4g.11186623.0.0.1e525d8cdvgi2s&file=aliyun-cli-linux-latest-amd64.tgz)

```shell
$ tar -xf aliyun-cli-linux-latest-amd64.tgz
$ cp aliyun /usr/bin/

在GitHub上下载您所需版本的阿里云CLI
https://github.com/aliyun/aliyun-cli/releases?spm=a2c4g.11186623.0.0.1e525d8cdvgi2s
```

配置：

```shell
# 交互式配置：
$ aliyun configure --profile aktest
Configuring profile 'aktest' in 'AK' authenticate mode...
Access Key Id []: xxxxxxxxxx
Access Key Secret []: xxxxxxxxxxxx
Default Region Id []: cn-hangzhou
Default Output Format [json]: json (Only support json)
Default Language [zh|en] en: 
Saving profile[aktest] ...Done

--profile: 指定配置名称。如果指定的配置存在，则修改配置。若不存在，则创建配置。

$ aliyun configure list     # 查看
$ aliyun configure delete--profile aktest    # 删除配置文件
```

```shell
# 非交互式配置
$ aliyun configure set --profile aktest --region cn-hangzhou --access-key-id xxxxx --access-key-secret xxxxxxxx
```

```shell
$ aliyun --help           # 查看帮助
```



```shell
- 获取账户余额

子账号授权： AliyunBSSReadOnlyAccess

$ aliyun bssopenapi QueryAccountBalance
{
	"Code": "200",
	"Data": {
		"AvailableAmount": "xxxx",
		"AvailableCashAmount": "xxxx",
		"CreditAmount": "0.00",
		"Currency": "CNY",
		"MybankCreditAmount": "0.00"
	},
	"Message": "success",
	"RequestId": "Cxxxx8B7-xxxx-xxxx-xxxx-DFBxxxxCE89B",
	"Success": true
}
```



### 2.SDK

[阿里云交易和账单管理API](https://help.aliyun.com/product/87964.html)

首先先在控制台的RAM访问控制中创建一个用户，提供[AliyunBSSReadOnlyAccess](https://ram.console.aliyun.com/policies/AliyunBSSReadOnlyAccess/System/content)权限，并将AccessKeyID和AccessKeySecret记录下来。随后将其替换下面这段代码中对应位置，就可以得到账户余额了。

```python
$ vim test.py
import base64
import datetime
import hmac
import json
import urllib
from hashlib import sha1
from random import choice
 
import requests
 
ACCESS_ID = "[你的AccessKey ID]"
SIGNATURE_KEY = "[你的AccessKey Secret]" + "&"
URL = "http://business.aliyuncs.com"
 
param = dict()
param["Action"] = "QueryAccountBalance"
param["Format"] = "JSON"
param["Version"] = "2017-12-14"
param["AccessKeyId"] = ACCESS_ID
param["SignatureMethod"] = "HMAC-SHA1"
param["Timestamp"] = datetime.datetime.utcnow().strftime("%Y-%m-%dT%H:%M:%SZ")
param["SignatureVersion"] = "1.0"
param["SignatureNonce"] = "".join([choice("0123456789ABCDEF") for _ in range(16)])
 
param_str = "&".join(
    [
        "{}={}".format(key, urllib.parse.quote(param[key], "-_.~"))
        for key in sorted(param)
    ]
)
 
sign_str = (
    urllib.parse.quote("POST", "-_.~")
    + "&"
    + urllib.parse.quote("/", "-_.~")
    + "&"
    + urllib.parse.quote(param_str, "-_.~")
)
 
signature = base64.b64encode(
    hmac.new(SIGNATURE_KEY.encode(), sign_str.encode(), sha1).digest()
).decode()
param["Signature"] = signature
 
req = requests.post(url=URL, data=param)
print(json.loads(req.content.decode())["Data"]["AvailableCashAmount"])
```

```shell
$ python3 test.py
```

---



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

---



## 3.7牛云

qshell安装配置

[下载安装包](https://devtools.qiniu.com/qshell-v2.10.0-linux-386.tar.gz?ref=developer.qiniu.com&s_path=%2Fkodo%2F1302%2Fqshell)

```shell
Usage:
  qshell account [<AccessKey> <SecretKey> <UserName>] [flags]
  
$ qshell account ak sk name

# qshell只能对存储空间操作
```



---



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



## 5.namesilo

https://www.namesilo.com/api-reference

```
访问：
https://www.namesilo.com/api/listDomains?version=1&type=xml&key=f338589c2756a2b44f931&withBid=1&pageSize=1400&skipExpired=1
```



脚本：

```python
import requests
import re
import time
import os

token = "5429948026:AAFtWaW1ZcU4IUNaNNtIFfK9qHIYoMjAlgg"
chat_id = "-613903645"

def gethtml():
    url = "https://www.namesilo.com/api/listDomains?version=1&type=xml&key=f338589c2756a2b44f931&withBid=1&skipExpired=1"
    html = requests.get(url).content
    html = str(html, "utf-8")
    return html

def sendtelegram(messages):
    url = "https://api.telegram.org/bot{}/sendMessage?chat_id={}&text={}".format(token, chat_id, str(messages))
    requests.get(url)


html = gethtml()

res = re.findall('expires="(.*?)">(.*?)</domain>', html)
for i in res:
    domain = i[1]
    domain_expire = i[0]
    domain_expire1 = time.strptime(domain_expire, "%Y-%m-%d")   # 获取结构化时间
    expire_timestamp = time.mktime(domain_expire1)              # 将结构化时间转换为时间戳
    now_timestamp = time.time()                                 # 当前时间戳
    cha = int(expire_timestamp) - int(now_timestamp)
    days = int(cha / 3600 / 24)
    if days < 10:           # 域名过期时间小于10天告警
        message = """=====  域 名 即 将 到 期 报 警  =====

    域名平台：namesilo
    域名: {0}
    域名过期时间: {1}
    剩余天数: {2}
        """.format(domain, domain_expire, days)
        sendtelegram(message)
```



输出到文件告警

```python
...................
    if days < 10:
        now_date = time.strftime("%Y-%m-%d", time.localtime())    # 当前格式化时间
        filename = now_date + ".txt"
        os.system("echo {}  {} >> {}".format(domain, domain_expire, filename))

os.system('curl -F chat_id={} -F document=@"{}" https://api.telegram.org/bot{}/sendDocument'.format(chat_id, filename, token))
messages = "namesilo平台,10天内即将过期域名列表，请及时更新！！！"
sendtelegram(messages)
```

