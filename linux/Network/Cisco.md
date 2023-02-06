## 1.交换机，路由器配置

### 1.交换机工作模式

```shell
Switch>用户模式
Switch>enable
Switch#特权模式
Switch#configure  terminal
Switch(config)#全局配置模式
Switch(config)#interface fastEthernet 0/1
Switch(config-if)#接口模式
exit返回上一模式
end直接退到特权模式
```

### 2.配置的准备工作

```shell
1. 空闲一段时间后，重回初始界面的问题
switch(config)#line con 0
switch(config-line)#exec-timeout 0 0
 
2.控制台消息打断输入的处理
switch(config)#line con 0
switch(config-line)#logging synchronous

3.禁用DNS查询
switch(config)#no ip domain-lookup
接口模式：shutdown          #禁用此接口
```

### 3.交换机的基本配置

```shell
Switch(config)#hostname SW1          # 修改主机名为SW1
Switch#show  running-config          # 查看配置信息
Switch#show version                  # 查看IOS版本信息
配置enable明文口令
全局配置模式：enable  password  123
配置enable加密口令
全局配置模式： enable  secret  456
配置Console口令
全局模式:line  console  0
password  789
Login
保存交换机的配置
特权： copy  running-config  startup-config
或 write
恢复设备出厂默认值
特权：erase  startup-config
      reload
```

```shell
1）查看MAC地址表
特权：show mac-address-table

2）使用CDP协议
用来查看邻居Cisco设备的信息
特权：show  cdp  neighbors

3）接口的工作模式配置
指定接口的双工模式
接口模式：duplex  {full | half | auto}
full（全双工）、Half(半双工)、Auto(自动协商)
指定接口的通信速率
接口模式：speed {10 | 100 | 1000 | auto}
查看以太网接口的双工模式和通信速率
特权：show interface fastethernet 0/24
 
4）配置管理用IP地址
Console不是唯一的管理手段，有时需要通过网络对设备进行远程管理
配置管理用IP地址
全局配置模式：interface  vlan  1
ip  address  192.168.1.100  255.255.255.0
no  shutdown

5）配置交换机默认网关
全局配置模式：
ip  default-gateway  192.168.1.254


```

```shell
telnet远程管理操作
1、远程管理交换机或路由器
1)                  配置交换机管理IP
2)                  全局: line  vty  0  4
              password 123
              login
3)                  全局模式配置明文或密文密码之一
4)                  配置客户机IP并telnet  交换机管理IP
 
2、远程管理路由器的配置不同之处是给路由器的接口配置IP其他都相同。
```

### 4.Vlan的配置

```shell
创建VLAN有两种方法：
1、全局配置模式创建vlan
全局：vlan  2     # 创建vlan2
Name  名字        #（给vlan2命名）

2、VLAN数据库配置模式
特权：vlan database
Vlan  2  name caiwu    # (创建vlan2并命名为caiwu)

3、删除vlan
进入vlan数据库或全局模式：no vlan 2

4、接口加入vlan
1）进入将要加入vlan的接口然后输入
switchport access vlan 3
2）、同时将多个接口加入vlan
全局： interface range f0/1 – 10
switchport access vlan  2     # 将1-10口同时加入vlan2

5、查看vlan信息
特权：show  vlan  brief

6、description  添加vlan描述信息
```

### 5.trunk的配置

```shell
1. 配置trunk
接口模式：switchport  mode  trunk(直接配置为trunk)
dynamic desirable   # (配置为动态企望)
dynamic auto        #（动态自动）
access              #（配置为接入链路）

2.在trunk链路上 移除某vlan
进入trunk接口：
switchport trunk allowed vlan remove 3    # 中继链路不允许传送vlan 3的数据

3.在trunk链路上 添加某vlan
进入trunk接口：
switchport trunk allowed vlan  add  3

4.查看接口模式
特权：show  interface  f0/5  switchport
```

### 6.EthernetChannel（以太网通道）

```shell
1、功能：多条线路负载均衡，带宽提高
容错，当一条线路失效时，其他线路通信，不会丢包

2、以太网通道的配置：
全局：interface range f0/6 – 8
switchport mode trunk
channel-group 1 mode on

1、查看以太网通道的配置：
特权：show etherchannel summary
2、以太网道必须遵循以下一些规则：
1）参与捆绑的端口必须属于同一个vlan,如果是在中继模式下，要求所有参加捆绑的端口配置成相同的中继模式。
2）所有参与捆绑的端口的物理参数设置必须相同，应该有同样的速度和全/半双工模式设置。
```

### 7.在路由器上配置DHCP服务

```shell
1、全局: ip dhcp pool  名字                #（定义地址池）
2、network 192.168.1.0 255.255.255.0      #（动态分配IP地址段）
3、default-router 192.168.1.254           #（动态分配的网关地址）
4、dns-server 202.106.0.20（              #动态分配的DNS服务器地址）此命令后可以跟多个备用的DNS地址。
5、全局：ip dhcp excluded-address 192.168.1.1   #（预留已静态分配的IP地址）
```

### 8.单臂路由

```shell
    单臂路由，即在路由器上设置多个逻辑子接口，每个子接口对应一个vlan。在每个子接口的数据在物理链路上传递都要标记封装。Cisco设备支持ISL和802.1q（dot1Q）协议。华为只支持802.1q。

Router>enable
Router#configure terminal
Router(config)#int fastEthernet 0/1.10           （进入子接口）
Router(config-subif)#encapsulation dot1Q 10      （封装dot1Q协议，建立与vlan10的关联）
Router(config-subif)#ip add 192.168.10.254 255.255.255.0
Router(config-subif)#exit
Router(config-if)#int f 0/1.20
Router(config-subif)#encapsulation dot1Q 20       （封装dot1Q协议，建立与vlan20的关联）
Router(config-subif)#ip add 192.168.20.254 255.255.255.0
Router(config-subif)#int f 0/1
Router(config-if)#no shutdown                     （启动f0/1的接口，包括所有子接口）
```

## 2.三层交换和路由配置

### 1.静态路由

```shell
1）、特点：
由管理员手工配置的，是单向的，因此需要在两个网络之间的边缘路由器上需要双方对指，否则就会造成流量有去无回，缺乏灵活性，适用于小型网络。
2)、配置
全局模式：
ip  route  目标网络ID  子网掩码  下一跳IP
3)、浮动路由
配置浮动静态路由，需设置管理距离大于1，从而成为备份路由，实现链路备份的作用。

### 默认路由
https://blog.csdn.net/weixin_55988897/article/details/126910092
```

### 2.动态路由

```shell
1、动态路由特点 
减少了管理任务 
占用了网络带宽
一、路由协议分类：
1、按应用范围的不同，路由协议可分为两类：
1）在一个AS内的路由协议称为内部网关协议（interior gateway protocol），正在使用的内部网关路由协议有以下几种：RIP-1，RIP-2，IGRP，EIGRP，IS-IS和OSPF。
2）AS之间的路由协议称为外部网关协议（exterior gateway protocol）。
外部网关协议（External Gateway Protocol，EGP，也叫域 间路由协议）。域间路由协议有两种：外部网关协议（EGP）和边界网关协议（BGP）

注：AS自治系统（Autonomous System，指一个互连网络，就是把整个Internet划分为许多较小的网络单位，这些小的网络有权自主地决定在本系统中应采用何种路由协议）

3）以情况下，需要使用BGP：
· 当你需要从一个AS发送流量到另一个AS时；
------------------------------------------------
2、按照路由执行的算法动态路由协议的分类 
1）距离矢量路由协议 
依据从源网络到目标网络所经过的路由器的个数选择路由 
RIP、IGRP
2）链路状态路由协议 
综合考虑从源网络到目标网络的各条路径的情况选择路由 
OSPF、IS-IS 
二、RIP路由协议
RIP是距离-矢量路由选择协议 
RIP度量值为跳数 ，最大跳数为15跳，16跳为不可达
RIP更新时间，每隔30s发送路由更新消息，UDP 520端口
RIP路由更新消息，发送整个路由表信息
------------------------------------------------
3、RIP v2的配置
全局：router  rip
version  2
no  auto-summary(关闭路由汇总)
network  主网络ID
```

### 3.三层交换技术

```shell
1、作用
使用三层交换技术实现VLAN间通信 
三层交换=二层交换+三层转发
2、基于CEF 的快速转发
主要包含两个转发用的信息表：
1）转发信息库（FIB）：FIB类似于路由表，包含路由表中转发信息的镜像。当网络的拓扑发生变化时，路由表将被更新，而FIB也将随之变化。
2）邻接关系表：每个FIB条目，邻接关系表中都包含相应的第2层地址。
3、虚拟接口（SVI）
三层交换机上配置的VLAN接口为虚接口
4、三层交换机的配置
1）、在三层交换机启用路由功能 
全局：ip  routing
2）、配置虚拟接口的IP 地址
全局：interface  vlan  2
ip  address  192.168.2.254  255.255.255.0 
no  shutdown 
3）、在三层交换机上配置Trunk并指定接口封装为802.1q
接口模式：switchport  trunk  encapsulation  dot1q 
switchport  mode  trunk
4）、配置路由接口
进入接口：no  switchport 
```



