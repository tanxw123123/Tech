https://lxnchan.cn/netmaker.html

https://docs.netmaker.org/getting-started.html  官方文档

Netmaker 是创建和管理虚拟 overlay 网络的工具

Netmaker 的核心是管理跨机器的 WireGuard，以创建合理的网络

## 1.准备

官方文档中对Netmaker服务端要求如下：

1.静态公网ip

2.最低1C1G（推荐2C4G）

3.Ubuntu 20.04

另外，还要求有一个能更改DNS记录的域名

```
oball48.com
```



开通防火墙端口：

| 类型    | 端口   | 作用                                          |
| ------- | ------ | --------------------------------------------- |
| TCP     | 80     | 申请R3证书                                    |
| TCP     | 443    | 控制台、API、gRPC                             |
| UDP     | 51820- | 业务端口，从51821开始，你想有几条隧道就到多少 |
| TCP+UDP | 53     | CoreDNS（可选）                               |



## 2.安装

- 官方nm-quick.sh脚本
- 自定义配置参数进行部署



### 一键安装

```shell
$ sudo wget -qO /root/nm-quick-interactive.sh https://raw.githubusercontent.com/gravitl/netmaker/master/scripts/nm-quick-interactive.sh && sudo chmod +x /root/nm-quick-interactive.sh && sudo /root/nm-quick-interactive.sh
```



```shell
# 解析域名
api.netmaker.oball48.com
broker.netmaker.oball48.com
dashboard.netmaker.oball48.com
netmaker.oball48.com
```

配置：

```
Access key: ba151c689ac88679

Access token: eyJhcGljb25uc3RyaW5nIjoiYXBpLm5ldG1ha2VyLm9iYWxsNDguY29tOjQ0MyIsIm5ldHdvcmsiOiJ3c2R5LXRlc3QiLCJrZXkiOiJiYTE1MWM2ODlhYzg4Njc5IiwibG9jYWxyYW5nZSI6IiJ9

join command: netclient join -t eyJhcGljb25uc3RyaW5nIjoiYXBpLm5ldG1ha2VyLm9iYWxsNDguY29tOjQ0MyIsIm5ldHdvcmsiOiJ3c2R5LXRlc3QiLCJrZXkiOiJiYTE1MWM2ODlhYzg4Njc5IiwibG9jYWxyYW5nZSI6IiJ9

docker: docker run -d --network host  --privileged -e TOKEN=eyJhcGljb25uc3RyaW5nIjoiYXBpLm5ldG1ha2VyLm9iYWxsNDguY29tOjQ0MyIsIm5ldHdvcmsiOiJ3c2R5LXRlc3QiLCJrZXkiOiJiYTE1MWM2ODlhYzg4Njc5IiwibG9jYWxyYW5nZSI6IiJ9 -v /etc/netclient:/etc/netclient --name netclient gravitl/netclient:v0.17.1

Join via Basic Auth: netclient join -n <network-name> -u <user name> -s <netmaker api domain>

Join via SSO: netclient join -n <network-name> -s <netmaker api domain>
```



访问： https://dashboard.netmaker.oball48.com/

## 3.入门

### 1.设置

1. 使用用户名和密码创建您的管理员用户。
2. 使用您的新用户登录
3. 单击创建网络创建您的第一个网络

### 2.创建网络

打开下方的 `UDP Hole Punching` udp打洞，其余保持默认。

![image-20230125145802154](D:\Tech\linux\Network\assets\image-20230125145802154.png)

### 3.创建密钥

将节点添加到网络通常需要密钥。

1. 单击 `Access Keys` 选项卡并选择您创建的网络。
2. 单击创建访问密钥
3. 给它起个名字（例如：“mykey”）和使用次数（例如：25）
4. 点击创建
5. 访问https://docs.netmaker.org/netclient.html#install在您的节点上安装 netclient。

### 4.部署节点

先决条件：您安装的每台机器都应该已经安装了 WireGuard

```shell
###
安装wireGuard

- windows：
https://download.wireguard.com/windows-client/wireguard-installer.exe

- linux：
$ apt update
$ apt install wireguard
```

然后，安装 [netclient](https://docs.netmaker.org/netclient.html#installation)

```shell
## linux(ubuntu):
$ curl -sL 'https://apt.netmaker.org/gpg.key' | sudo tee /etc/apt/trusted.gpg.d/netclient.asc
$ curl -sL 'https://apt.netmaker.org/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/netclient.list
$ apt update
$ apt install netclient
$ netclient join -t token值
```

在所有节点上安装后，您可以通过从任何其他节点 ping 任何节点的私有地址来测试连接

```shell
$ ping 10.20.30.254
```

### 5.管理节点

您的机器现在应该在控制面板中可见。

![image-20230125152856976](D:\Tech\linux\Network\assets\image-20230125152856976.png)

您可以通过在 NODES 选项卡中选择它来查看/修改/删除任何节点

### 6.卸载网络客户端

```shell
# 请在每个节点上运行以下命令：（将默认替换为网络的实际名称，例如 wg-net）
$ sudo netclient leave -n default
```



## 4.网络客户端配置

### 1.安装

在将机器添加到网络之前，必须安装 netclient。成功的安装会在机器上设置 netclient 可执行文件并将其添加为系统守护进程。守护进程将监听它加入的任何网络的变化。

客户端安装**不会**将客户端添加为任何网络的成员。安装客户端后，您必须运行：

```shell
> netclient join -t <token>
```

### 2.管理网络客户端

连接/断开网络：（将默认替换为网络的实际名称）

```shell
> netclient connect -n network
> netclient disconnect -n network
```

您还可以从 UI 断开连接并重新连接。单击要断开/重新连接的节点，然后单击编辑。

列出网络：

```shell
netclient list | jq
```



## 5.外部客户端

`Netclient` 目前只支持 Linux、macOS 和 Windows，如果 Android 和 iOS 端想要加入 VPN 私有网络，只能通过 WireGuard 原生客户端来进行连接。要想做到这一点，



```shell
1. 在 Dashboard 首页点击 Nodes ，然后找到你想作为出口的 Node ，再找到 Ingress Status 列，点击后会显示询问是否将该 Node 作为出口，点接受后查看对应列是否有变成绿色对号即可。
2. 打开侧边栏，选择 `External Client`，上方选择 Mesh 网络，然后找到某一出口 Node，点击对应行的加号，右侧 Client就会新增一行，此时再按照不同操作系统配置即可。一个Client对应一个客户端。
3. WireGuard 客户端可以下载该配置文件或者扫描二维码进行连接。
```

