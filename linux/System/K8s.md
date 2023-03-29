## 1.快速安装docker

```shell
# centos && ubuntu
$ curl -sSL https://get.daocloud.io/docker | sh
```

## 2.k8s组件

**`i. master节点组件：`**

- *API Server*：提供Kubernetes API接口，主要处理 Rest操作以及更新Etcd中的对象。是所有资源增删改查的唯一入口。
- *Scheduler*：资源调度，绑定Pod到Node上；
- *Controller*: 负责运行控制器进程。负责维护群集的状态，比如故障检测、自动扩展、滚动更新等；
  - `deployment`：管理pod多副本
  - `replicaSet`：pod的多副本，创建deployment会自动创建replicaSet，通常都用deployment
  - `daemonSet`：每个节点运行一个pod
  - `statefuleSet`：保证pod在整个生命周期中名称不变，其他conntroller在pod发生故障需要删除并重启时，pod的名称会发生改变。
  - `Job`：运行结束就删除的应用，其他controller中的pod会长期持续的运行。
- *etcd*：Kubernetes 提供的一个高可用的键值数据库，用于保存集群所有的网络配置和资源对象的状态信息，也就是保存了整个集群的状态。



**`ii. node节点组件：`**

- *kubelet*：负责pod的生命周期管理，维护和管理该Node上面的所有容器；
- *kube-proxy*：是实现service的通信与负载均衡机制的重要组件，将到service的请求转发到后端的pod上；
- *docker*：是负责容器的创建和管理工作；
- *pod*：k8s的最小工作单元，一个pod包含一个或多个容器。pod作为整体被master调度到某个node上运行，pod中所有容器使用相同的ip和端口。

## 3.k8s集群搭建

### 1.kuboard spray

**！！！** 重要： kuboard-spray 所在机器不能当做 K8S 集群的一个节点，因为安装过程中会重启集群节点的容器引擎，这会导致 kuboard-spray 被重启掉。



**`安装kuboard spray`**

```shell
$ docker run -d \
  --privileged \
  --restart=unless-stopped \
  --name=kuboard-spray \
  -p 80:80/tcp \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v ~/kuboard-spray-data:/data \
  eipwork/kuboard-spray:latest-amd64
  
# 在浏览器打开地址 http://这台机器的IP，用户名：admin  输入默认密码 Kuboard123，即可登录 Kuboard-Spray 界面
```



**`导入资源包`**

![image-20230124161331546](D:\Tech\linux\System\assets\image-20230124161331546.png)



**`添加集群安装计划`**

![image-20230124163117853](D:\Tech\linux\System\assets\image-20230124163117853.png)

全局配置：用户名，密码，端口

容器引擎选择： docker_20.10

Kubenetes在1.24 版本里弃用docker

![image-20230124163638504](D:\Tech\linux\System\assets\image-20230124163638504.png)

配置节点：

`control-plane`节点过去叫`master`节点，由于"某些原因"，现在开源社区很多项目都开始避免使用`master`这个词.

![image-20230124165138624](D:\Tech\linux\System\assets\image-20230124165138624.png)



**`安装`**

![image-20230124165433668](D:\Tech\linux\System\assets\image-20230124165433668.png)



### 2.二进制安装

```shell

```



## 4.Pod管理

### 1.创建pod

```shell
- 命令行

$ kubectl run nginx --image=nginx    # 创建pod
$ kubectl delete pod nginx           # 删除pod

# kubectl run --help  查看帮助
```

```yaml
- yaml方式

$ kubectl run nginx --image=nginx --image-pull-policy=IfNotPresent --dry-run=server -o yaml

# --dry-run 模拟运行，并不会真的创建一个pod ， --dry-run=client输出信息少 ，--dry-run=server输出信息多， -o yaml以yaml文件的格式输出

$ kubectl create deployment nginx --image=nginx --dry-run=client -o yaml      # 生成一个deployment的yaml文件
apiVersion: apps/v1     # <---  apiVersion 是当前配置格式的版本
kind: Deployment     #<--- kind 是要创建的资源类型，这里是 Deployment
metadata:        #<--- metadata 是该资源的元数据，name 是必需的元数据项
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:        #<---    spec 部分是该 Deployment 的规格说明
  replicas: 1        #<---  replicas 指明副本数量，默认为 1
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:        #<---   template 定义 Pod 的模板，这是配置文件的重要部分
    metadata:        #<---     metadata 定义 Pod 的元数据，至少要定义一个 label。label 的 key 和 value 可以任意指定
      creationTimestamp: null
      labels:
        app: nginx
    spec:           #<---  spec 描述 Pod 的规格，此部分定义 Pod 中每一个容器的属性，name 和 image 是必需的
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

基于上面的deployment服务生成service的yaml配置

```shell
# 基于上面的deployment服务生成service的yaml配置
$ kubectl expose deployment nginx --port=80 --target-port=80 --dry-run=client -o yaml
$ kubectl expose deployment nginx --port=80 --target-port=80 --type=NodePort --dry-run=client -o yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
status:
  loadBalancer: {}
```

```shell
# 伸缩副本
$ kubectl scale deployment nginx --replicas=3
```

 重启pod名包含“nginx”的多个pod

```shell
$ kubectl get pod -n default | grep nginx | awk '{print $1}' | xargs kubectl delete pod -n default
```

过滤包含“nginx”的pod

```shell
$ kubectl get pod --all-namespaces | grep "nginx"
```

###  2.pod健康检查

`1.livenessProbe探针（存活检查）`

根据用户自定义规则来判定pod是否健康，kubelet会根据其重启策略来决定是否重启

**pod重启策略：**

- Always: 当容器终止退出后，总是重启容器，默认策略。

- onFailure: 当容器异常退出（退出状态码非0）时，才重启容器
- Never：当容器终止退出，从不重启容器

```yaml
例子1： 定义存活命令

apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
      
$ kubectl describe pod xxxxx

```

`2.ReadinessProbe探针（就绪检查）`

根据用户自定义规则来判断pod是否健康，如果探测失败，控制器会将此pod从对应service的endpoint列表中移除，从此不再将任何请求调度到此Pod上，直到下次探测成功。

```yaml
# 这个探针执行ls /ready命令，如果这个文件存在，则返回0，说明Pod就绪了，否则返回其他状态码
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:alpine
        name: container-0
        resources:
          limits:
            cpu: 100m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        readinessProbe:      # Readiness Probe
          exec:              # 定义 ls /ready 命令
            command:
            - ls
            - /ready
      imagePullSecrets:
      - name: default-secret

$ kubectl apply -f xxx.yaml

# 这里由于nginx镜像不包含/ready这个文件，所以在创建完成后容器不在Ready状态，如下所示，注意READY这一列的值为0/1，表示容器没有Ready。
$ kubectl get pod
NAME                     READY     STATUS    RESTARTS   AGE
nginx-7955fd7786-686hp   0/1       Running   0          7s
nginx-7955fd7786-9tgwq   0/1       Running   0          7s
nginx-7955fd7786-bqsbj   0/1       Running   0          7s

# 创建Service:
apiVersion: v1
kind: Service
metadata:
  name: nginx        
spec:
  selector:          
    app: nginx
  ports:
  - name: service0
    targetPort: 80   
    port: 8080       
    protocol: TCP    
  type: ClusterIP
  
###
$ kubectl describe svc nginx
Name:              nginx
......
Endpoints:         
......

###
如果此时给容器中创建一个/ready的文件，让Readiness Probe成功，则容器会处于Ready状态。再查看Pod和Endpoints，发现创建了/ready文件的容器已经Ready，Endpoints也已经添加
$ kubectl exec nginx-7955fd7786-686hp -- touch /ready

$ kubectl get po -o wide
NAME                     READY     STATUS    RESTARTS   AGE       IP
nginx-7955fd7786-686hp   1/1       Running   0          10m       192.168.93.169 
nginx-7955fd7786-9tgwq   0/1       Running   0          10m       192.168.166.130
nginx-7955fd7786-bqsbj   0/1       Running   0          10m       192.168.252.160

$ kubectl get endpoints
NAME       ENDPOINTS           AGE
nginx      192.168.93.169:80   14d

```

**上面两种探针都支持3种探测方法：**

- Exec： 通过执行命令的方式来检查服务是否正常
- Httpget： 通过发送http/htps请求检查服务是否正常，返回的状态码为200-399则表示容器健康
- tcpSocket： 通过容器的IP和Port执行TCP检查，如果能够建立TCP连接，则表明容器健康，这种方式与HTTPget的探测机制有些类似，tcpsocket健康检查适用于TCP业务。





### 3.pod调度方式

```
https://cloud.tencent.com/developer/article/1644857

Pod调度策略除了系统默认的kube-scheduler调度器外还有以下几种实现方式：

1.nodeName（直接指定Node主机名）

2.nodeSelector （节点选择器，为Node打上标签，然后Pod中通过nodeSelector选择打上标签的Node）

3.污点taint与容忍度tolerations

4.NodeAffinity 节点亲和性

5.PodAffinity Pod亲和性

6.PodAntAffinity Pod反亲和性
```



```shell
一、 kube-scheduler调度介绍
默认调度器。
满足一个Pod调度请求的所有Node称之为可调度节点
如果没有任何一个Node能满足Pod的资源请求，那么这个Pod将一直停留在未调度状态直到调度器能够找到合适的Node

两个步骤：
1.过滤阶段： 过滤出可调度节点
2.打分阶段： 对可调度节点进行打分

二、Pod调度之nodeName
nodeNamed这种调度方式比较简单，我们可以指定Pod在哪台Node上进行运行，通过spec.nodeName参数来指定Node主机名称即可。

spec:
#指定该Pod运行在k8s-node02节点上
  nodeName: k8s-node02
  
三、Pod调度之nodeSelector
nodeSelector用于将Pod调度到匹配Label的Node上,所以要先给node打上标签


https://www.jianshu.com/p/2d59e83162a7  给节点打标签
- 添加标签
- 查看标签
spec:
  nodeSelector:                      #指定标签选择器
    team: development                #label指定开发团队的label
    

四、Pod调度之污点与容忍
污点（taint）： 定义在Node之上的键值型的数据。用于让节点拒绝将Pod调度运行于其上，除非该Pod对象具有接纳Node污点的容忍度
容忍度（tolerations）： 定义在Pod对象的键值型属性数据，用于·配置其可容忍的Node污点，而且调度器仅能将Pod对象调度至其能够容忍该Node污点的Node之上。

污点类型:
• NoSchedule ：为Node添加污点等级为NoSchedule,除容忍此污点的Pod以外的其它Pod将不再被调度到本机。
•PreferNoSchedule：为Node添加污点等级为PreferNoSchedule,不能容忍此污点的Pod对象尽量不要调度到当前节点，如果没有其它节点可以供Pod选择时，也会接受没有容忍此污点的Pod对象。
•NoExecute：为Node添加污点等级为NoExecute，能容忍此污点的Pod将被调度到此节点，而且节点上现存的Pod对象因节点使用了NoExceute等级的污点，则现存的Pod将被驱赶至其它满足条件的Node

# 添加污点： 为k8s-node02添加污点，污点程度为NoSchedule，type=calculate为标签
$ kubectl taint node k8s-node02 type=calculate:NoSchedule

# 查看污点：
$ kubectl describe nodes k8s-node02 | grep Taints

# 删除污点：
$ kubectl taint node k8s-node02 type:NoSchedule-


容忍度介绍及定义

```

### 4.pod间通信

https://www.51cto.com/article/702401.html  网络模型

#### 1.Docker容器间通信

我们安装Docker时，它会自动创建三个网络，bridge（创建容器默认连接到此网络）、 none 、host；[网址](https://cloud.tencent.com/developer/article/1674259)

```shell
- host：容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。相当于Vmware中的桥接模式
- None：该模式关闭了容器的网络功能，相当于一个回环网络。
- Bridge：此模式会为每一个容器分配、设置IP等，并将容器连接到一个叫docker0的虚拟网桥，通过docker0网桥以及Iptables nat表配置与宿主机通信。相当于Vmware中的NAT模式

- overlay：顾名思义：覆盖，但它又不是覆盖，它的作用就是在容器原有的网络基础之上，再添加一块网卡，并为其分配一个IP地址，可以将所有的docker容器关联到同一个局域网中，适用于容器与容器是跨主机进行通信的场景。
```

```shell
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
35ac75169514   bridge    bridge    local
20310e4d27d0   host      host      local
7376d0c6efe5   none      null      local
```

**Bridge模式**

启动docker--》创建docker0虚拟网桥（类似交换机）

容器连接到docker0虚拟网桥--》然后，分配ip给容器



**单机环境下的网络拓扑如下（主机地址是10.10.0.186/24）：**

![image-20230109192801436](D:\Tech\linux\System\.assets\image-20230109192801436.png)



```shell
实现的效果如下：
 
1. 基于docker0网络创建2个容器，分别是test1、test2。
2. 创建自定义网络，网络类型为bridge，名称为my_net1.基于此网络创建两个容器test3,test4（不指定网段）
3. 创建自定义网络，网络类型为bridge，名称为my_net2，指定网段为172.20.18.0/24，基于此网络创建两个容器test5(ip为172.20.18.5)，test6（IP为172.20.18.6）。

配置实现test2能够和test3相互通信，test4和test5可以相互通信。
```

配置如下：

```shell
$ docker run -itd --name test1 busybox  #创建一个容器test1
$ docker run -itd --name test2 busybox  # 同上

$ docker network create -d bridge my_net1             #创建一个桥接网络，名称为my_net1
$ docker run -itd --name test3 --network my_net1 busybox       #基于my_net1创建容器test3
$ docker run -itd --name test4 --network my_net1 busybox       # 同上

$ docker network create -d bridge --subnet 172.20.18.0/24 my_net2    # 创建一个桥接网络my_net2，并指定其网段
$ docker run -itd --name test5 --network my_net2 --ip 172.20.18.5 busybox    #基于my_net2网络，创建一个容器test5，并且指定其IP地址
$ docker run -itd --name test6 --network my_net2 --ip 172.20.18.6 busybox    # 同上

$ docker network connect my_net1 test2          #将test2连接到my_net1这个网络，相当于将test2这个容器添加了一块网卡，然后连接到了my_net1这个交换机 
$ docker exec test2 ping test3           #进行ping测试，可以发现test2 可以ping通test3了。
$ docker exec test2 ip a s            # 查看此时容器ip

$ docker network connect my_net2 test4    # 将test4连接到my_net2网络
$ docker exec test4 ping test5 
```

---



#### 2.Docker容器跨主机通信

##### 1.直接路由方式

```shell
https://cloud.tencent.com/developer/article/1587094
# 避免不同主机上的容器使用了相同的IP，为此我们应该为不同的主机分配不同的子网来保证.

各项配置如下：

- 主机1的IP地址为：192.168.254.71
- 主机2的IP地址为：192.168.254.72
- 为主机1上的Docker容器分配的子网：10.0.128.0/24
- 为主机2上的Docker容器分配的子网：10.0.129.0/24
```

修改主机1和主机2的子网：

```shell
# host1
$ cat /etc/docker/daemon.json
{
  "bip":"10.0.128.1/24"
}

# host2
$ cat /etc/docker/daemon.json
{
  "bip":"10.0.129.1/24"
}

$ systemctl restart docker    # 重启容器
```

添加路由规则：

```shell
# host1
$ route add -net 10.0.129.0/24 gw 192.168.254.72

# host2
$ route add -net 10.0.128.0/24 gw 192.168.254.71
```



在主机1上，ping主机2的docker0地址

```shell
$ ping 10.0.129.1
PING 10.0.129.1 (10.0.129.1) 56(84) bytes of data.
64 bytes from 10.0.129.1: icmp_seq=1 ttl=64 time=0.770 ms
```

在主机2上，ping主机1的docker0地址

```shell
$ ping 10.0.128.1
PING 10.0.128.1 (10.0.128.1) 56(84) bytes of data.
64 bytes from 10.0.128.1: icmp_seq=1 ttl=64 time=0.426 ms
```

ok，既然docker0都通了，那么起一个docker容器，会不会也是通的的呢？测试一下

```shell
# host1
$ docker run -itd --name test1 busybox   # 容器ip为 10.0.128.2

# host2
$ docker run -itd --name test2 busybox   # 容器ip为 10.0.129.2
```



在主机1上的容器中 ping 主机2中的容器

先来ping 主机2的docker0，再ping 主机2中的容器

```shell
$ docker exec test1 ping 10.0.129.1
PING 10.0.129.1 (10.0.129.1): 56 data bytes
64 bytes from 10.0.129.1: seq=0 ttl=64 time=0.095 ms

$ docker exec test1 ping 10.0.129.2     # ping不通

```

从结果中，可以发现。docker0是通的，但是主机2中的容器是不通的，为什么呢？



配置iptables规则

```shell
# host1
$ iptables -t nat -I PREROUTING -s 10.0.128.0/24 -d 10.0.129.0/24 -j DNAT --to 10.0.128.1

- 当源地址为10.0.128.0/24网段 访问 10.0.129.0/24 时，在路由之前，将ip转换为10.0.128.1
# 注意：一定要加-d参数。如果不加，虽然docker之间可以互通，但是不能访问网站，比如百度，qq之类的！
# 为什么呢？访问10.0.129.0/24 时，通过docker0网卡出去的。但是访问百度，还是通过docker0，就出不去了！
# 真正连接外网的是eth0网卡，必须通过它才行！因此必须要指定-d参数！

# host2
$ iptables -t nat -I PREROUTING -s 10.0.129.0/24 -d 10.0.128.0/24 -j DNAT --to 10.0.129.1
```

---



##### 2.overlay

Overlay网络实际上是目前最主流的容器跨节点数据传输和路由方案。

环境：

- node1：192.168.254.60

- node2：192.168.254.203

- node3：192.168.254.71



`i.安装consul`

```shell
node1上：
$ docker run -d -p 8500:8500 -h consul --name consul progrium/consul -server -bootstrap

-p 8500:8500 指定映射端口8500
-h consul 指定主机名
--name consul 是容器的名字
progrium/consul 镜像名
-server 设置Agent是server模式
-bootstrap 设置服务为“bootstrap”模式
```

容器启动后，可以通过 [http://192.168.254.60:8500](http://192.168.254.60:8500/) 访问 Consul



```shell
node2、node3上： 
$ vim /etc/docker/daemon.json
{
  "hosts":["tcp://0.0.0.0:2375","unix:///var/run/docker.sock"],
  "cluster-store": "consul://192.168.xx.xx:8500",
  "cluster-advertise": "192.168.254.203:2375"      # node3就填自己的ip                  
}

host 开启和监听2375端口，同时使用docker.sock文件
--cluster-store指定 consul 的地址。
--cluster-advertise 告知 consul 自己的连接地址。

# 注意：记得修改docker.service.因为配置和daemon.json冲突
- 修改 /usr/lib/systemd/system/docker.service
- 将-H fd://去掉

$ vim /usr/lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd  --containerd=/run/containerd/containerd.sock
```

```shell
$ systemctl daemon-reload
$ systemctl restart docker
```

网页上访问： [http://192.168.xx.xx:8500](http://192.168.1.12:8500/)

在KEY/VALUE -> docker -> nodes 下如果有你创建的两个节点node2（192.168.254.203），node3（192.168.254.71），说明成功。

![image-20230111151405108](D:\Tech\linux\System\assets\image-20230111151405108.png)



`ii.创建overlay网络`

```shell
在 node2 中创建 overlay 网络 ol1：

$ docker network create -d overlay ol1

# 同时查看node3，也会出现ol1网络，这是因为创建 ol1 时 node2 将 overlay 网络信息存入了 consul，node3 从 consul 读取到了新网络的数据。之后 ol1 的任何变化都会同步到 node2 和 node3。
```



`iii.启动容器测试`

```shell
分别在node2和node3启动容器
$ docker run -itd --name test1 --network ol1 busybox   # node2节点
$ docker run -itd --name test2 --network ol1 busybox   # node3节点

$ docker exec test1 ping test2    # node2上ping
PING test2 (10.0.0.3): 56 data bytes
64 bytes from 10.0.0.3: seq=0 ttl=64 time=0.951 ms

能看到已经ping通了

```



- **docker 会创建一个 `bridge` 网络` “docker_gwbridge”`，为所有连接到 `overlay` 网络的容器提供访问外网的能力!!**



##### 3.容器网络插件

```shell
# 安装flannel网络插件(简单易用)

# 下载flannel插件的yml
$ wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 修改kube-flannel.yml中的镜像仓库地址为国内源
$ sed -i 's/quay.io/quay-mirror.qiniu.com/g' kube-flannel.yml

# 安装网络插件
$ kubectl apply -f kube-flannel.yml
```



```shell
# 下载calico插件的yaml
$ wget https://docs.projectcalico.org/manifests/calico.yaml
```



### 5.pod状态

Pending：pod 正在等待 kube-scheduler 选择合适的节点创建。

Running：pod 已正常创建，并且至少有一个容器正在运行。

Succeeded：所有容器已成功启动运行。

Failed：pod 的容器非正常退出。

Unknown：无法获取 pod 状态，可能节点间通信出现问题。

```shell
# 解决Terminating状态的Pod删不掉的问题
$ kubectl delete pod <pod> --grace-period=0 --force

--------------------------------------------
# k8s pod一直处于ContainerCreating状态
$ kubectl describe pod nginx-79f79f868d-876f4     #查看报错

$ kubectl get pod -o wide   # 查看对应pod所在节点
我这里直接重启节点恢复
```



### 6.创建pod流程

1.客户端提交 Pod 的配置信息（可以是 yaml 文件定义好的信息）到 kube-apiserver；
2.Apiserver 收到指令后，通知给 controller-manager 创建一个资源对象；
3.Controller-manager 通过 api-server 将 pod 的配置信息存储到 ETCD 数据中心中；

4.scheduler 查看 k8s api ，类似于通知机制。首先判断：pod.spec.Node == null? 若为null，表示这个Pod请求是新来的，需要创建；因此先进行调度计算，找到最`闲`的node。然后将信息在etcd数据库中更新分配结果。
5.随后目标节点的 kubelet 进程通过 api-server 提供的接口监测到 kube-scheduler 产生的 pod 绑定事件，然后从 etcd 获取 pod 清单，下载镜像并启动容器

### 7.外部访问pod的方式

kubernetes中对外暴露服务的方式有两种：

- service（NodePort或者外部LoadBalancer）- 四层的负载均衡
- ingress  - 七层负载均衡



hostNetwork、hostPort、NodePort、LoadBalancer、Ingress

```yaml
# hostPort
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
          hostPort: 8082     # 通过物理机的IP地址和8082端口访问
          
- 服务监听在容器的网络栈，宿主机在防火墙上做了转发，所以查询宿主机8080端口，发现并没有服务监听
- 查看防火墙，宿主机做了端口转发
```



```shell
# nodePort  4层负载均衡
NodePort 是Kubernetes中“Service”资源的一个属性，默认“Service”实现集群内服务访问，当增加“NodePort”时，服务便可以在集群外被访问到，Kubernetes默认会从宿主机的端口30000-32767之间选择一个来提供访问

“nodePort”方式的Service工作原理是这样：当流量进入宿主机暴露的30000端口后，会被转发给“Service”的“Cluster IP+端口”，然后通过Iptables（也有可能是ipvs，具体看实现）转发到对应的Pod。
```

#### 1.service

#### 2.Ingress

##### 1.介绍

```shell
# Ingress  7层负载均衡
- 使用Ingress暴露服务的方式在生产环境中使用的比较多
- K8s 并没有自带 Ingress Controller，它只是一种标准，具体实现有多种，需要自己单独安装，常用的是 Nginx Ingress Controller 和 Traefik Ingress Controller。

Ingress Controller 收到请求，匹配 Ingress 转发规则，匹配到了就转发到后端 Service，而 Service 可能代表的后端 Pod 有多个，选出一个转发到那个 Pod，最终由那个 Pod 处理请求。

Ingress可以和Ingress Controller不在同一namespace，但必须与声明的服务在同一namespace

---------------------------------------------------------------------------------
nginx-ingress 组成
- ingress controller：将新加入的Ingress转化成Nginx的配置文件并使之生效
- ingress服务：将Nginx的配置抽象成一个Ingress对象，每添加一个新的服务只需写一个新的Ingress的yaml文件即可

. ingress controller通过和kubernetes api交互，动态的去感知集群中ingress规则变化，然后读取它；
. 按照自定义的规则，规则就是写明了哪个域名对应哪个service，生成一段nginx配置，再写到nginx-ingress-control的pod里；
. 这个Ingress controller的pod里运行着一个Nginx服务，控制器会把生成的nginx配置写入/etc/nginx.conf文件中，然后reload一下使配置生效。以此达到域名分配置和动态更新的问题。

```

##### 2.部署ingress-controller

```shell
- 下载部署文件
$ wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/cloud/deploy.yaml

- 修改文件
$ vim deploy.yaml
spec:
  externalTrafficPolicy: Local
  ports:
  - appProtocol: http
    name: http
    port: 80
    protocol: TCP
    targetPort: http
    nodePort: 80    # 添加此行
  - appProtocol: https
    name: https
    port: 443
    protocol: TCP
    targetPort: https
    nodePort: 443   # 添加此行
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  type: NodePort   # 修改为NodePort
```

```shell
$ kubectl apply -f deploy.yaml
# 启动报错如下：
The Service "ingress-nginx-controller" is invalid: spec.ports[0].nodePort: Invalid value: 80: provided port is not in the valid range. The range of valid ports is 30000-32767

- 是因为k8s的node节点的端口默认被限制在30000-32767的范围

# 解决办法：
编辑 kube-apiserver.yaml文件
$ vim /etc/kubernetes/manifests/kube-apiserver.yaml
将spec.containers.command的最后面这一行，如下内容
- --service-node-port-range=30000-32767  修改为
- --service-node-port-range=1-65535

然后重启kubelet
$ systemctl restart kubelet
```

```shell

$ kubectl get pod -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-hcwhb        0/1     Completed   0          14m
ingress-nginx-admission-patch-zws8l         0/1     Completed   0          14m
ingress-nginx-controller-6bc476f787-7zsgw   1/1     Running     0          14m


#- 查看nginx-ingress服务，状态为pending,原因为未部署loadBalancer或者开启NodePort  #####上面已经修改了deploy.yaml, 此步骤忽略


$ kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                 AGE
ingress-nginx-controller             NodePort    10.233.71.167    <none>        80:80/TCP,443:443/TCP   15m
ingress-nginx-controller-admission   ClusterIP   10.233.111.147   <none>        443/TCP                 15m

- 安装完控制器成后会自动创建一个 名为 nginx 的 IngressClass 对象
$ kubectl get ingressclass
NAME    CONTROLLER             PARAMETERS   AGE
nginx   k8s.io/ingress-nginx   <none>       15m


$ curl 192.168.254.51
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

##### 3.创建ingress

```shell
$ kubectl create ingress --help       # 查看帮助
$ kubectl create ingress my-tomcat --class=nginx \
  --rule="foo.com/path*=svc:8080" \
  --rule="bar.com/admin*=svc2:http" --dry-run=client -o yaml > ingress-tomcat.yaml   #生成ingress的yaml文件
```

```yaml
# 修改文件
$ vim ingress-tomcat.yaml    
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: default
  name: my-tomcat
spec:
  ingressClassName: nginx    # 关联的 ingress-nginx 控制器,上面查询的 kubectl get ingressclass
  rules:
  - host: tomcat.com
    http:
      paths:
      - backend:
          service:
            name: tomcat
            port:
              number: 8080
        path: /
        pathType: Prefix
```

```shell
- 执行yaml文件
$ kubectl apply -f ingress-tomcat.yaml
$ kubectl get ingress    # 查看配置情况
NAME        CLASS   HOSTS        ADDRESS         PORTS   AGE
my-tomcat   nginx   tomcat.com   10.233.71.167   80      10m
```

```shell
- 查看controller的nginx配置文件：
$ kubectl exec -it ingress-nginx-controller-6bc476f787-7zsgw -n ingress-nginx /bin/bash

$ grep 'tomcat.com' nginx.conf
	## start server tomcat.com
		server_name tomcat.com ;
	## end server tomcat.com
```

##### 4.创建deploy服务

```yaml
--- 创建tomcat服务 ---

$ vim tomcat-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: tomcat
  name: tomcat-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tomcat
  template:
    metadata:
      labels:
        app: tomcat
    spec:
      containers:
      - image: tomcat:7.0.57-jre7
        name: tomcat
        ports:
        - containerPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: tomcat
  name: tomcat
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: tomcat
```

```shell
$ kubectl apply -f tomcat-deploy.yaml
```

```shell
- 由于我的域名不是公网解析的，所以我们修改本地hosts，以便访问

- 先查看controller运行在哪个节点
$ kubectl get pod -n ingress-nginx -o wide
NAME                                        READY   STATUS      RESTARTS   AGE   IP              NODE        NOMINATED NODE   READINESS GATES
ingress-nginx-admission-create-hcwhb        0/1     Completed   0          13m   10.234.24.77    k8s-node2   <none>           <none>
ingress-nginx-admission-patch-zws8l         0/1     Completed   0          13m   10.234.226.67   k8s-node1   <none>           <none>
ingress-nginx-controller-6bc476f787-7zsgw   1/1     Running     0          13m   10.234.24.78    k8s-node2   <none>           <none>

# 我们看到controller运行在node2节点，node2节点的ip是：192.168.254.53

- 本地windows hosts文件解析：
192.168.254.53 tomcat.com
```

访问：

![image-20230315211609212](D:\Tech\linux\System\assets\image-20230315211609212.png)

---



**创建一个nginx项目：**

```yaml
- 创建nginx的ingress文件

$ vim ingress-nginx.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: default
  name: my-nginx
spec:
  ingressClassName: nginx
  rules:
  - host: mynginx.com
    http:
      paths:
      - backend:
          service:
            name: nginx
            port:
              number: 80
        path: /
        pathType: Prefix
        
$ kubectl apply -f ingress-nginx.yaml        
```

```yaml
- 创建nginx的deployment文件
$ vim nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
                
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
    
$ kubectl apply -f nginx-deployment.yaml
```

```
- 最后在windows本地解析域名
192.168.254.53 mynginx.com
```

访问测试：

![image-20230316102218296](D:\Tech\linux\System\assets\image-20230316102218296.png)



##### 5.基于 TLS 的ingress服务

```shell
- 执行以下命令，生成TLS证书

$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=mynginx.com/O=mynginx.com"
```

```shell
- 执行以下命令根据生成的TLS证书文件创建集群的secret

$ kubectl create secret tls cert-example --key tls.key --cert tls.crt
```

```shell
- 执行以下命令，查看新建TLS证书配置

$ kubectl get secret cert-example
```

```yaml
- 配置ingress

$ cat ingress-nginx.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: default
  name: my-nginx
spec:
  tls:                       #
  - hosts:                   #
    - mynginx.com             #
    secretName: cert-example  #
  ingressClassName: nginx
  rules:
  - host: mynginx.com
    http:
      paths:
      - backend:
          service:
            name: nginx
            port:
              number: 80
        path: /
        pathType: Prefix
        
$ kubectl apply -f ingress-nginx.yaml        
```

```shell
$ kubectl apply -f nginx-deployment.yaml
```

```shell
# 访问https

$ curl -k -v https://mynginx.com
* About to connect() to mynginx.com port 443 (#0)
*   Trying 192.168.254.53...
* Connected to mynginx.com (192.168.254.53) port 443 (#0)
* Initializing NSS with certpath: sql:/etc/pki/nssdb
* skipping SSL peer certificate verification
* SSL connection using TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
* Server certificate:
* 	subject: O=mynginx.com,CN=mynginx.com
* 	start date: Mar 17 04:33:21 2023 GMT
* 	expire date: Mar 16 04:33:21 2024 GMT
* 	common name: mynginx.com
* 	issuer: O=mynginx.com,CN=mynginx.com
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: mynginx.com
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Fri, 17 Mar 2023 05:09:57 GMT
< Content-Type: text/html
< Content-Length: 615
< Connection: keep-alive
< Last-Modified: Tue, 13 Dec 2022 15:53:53 GMT
< ETag: "6398a011-267"
< Accept-Ranges: bytes
< Strict-Transport-Security: max-age=15724800; includeSubDomains
< 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>

....
```

### 8.pod报错

#### 1.kubelet 压力驱逐

![image-20230320170503937](D:\Tech\linux\System\assets\image-20230320170503937.png)

查看详细报错信息：

```shell
$ kubectl describe pod node-exporter-qxvnw -n monitoring
Warning  Evicted    2m55s  kubelet            The node had condition: [DiskPressure]

```

原因是 kubelet 检测到本地磁盘使用率超过了 85% ，这是 kubelet 的默认配置!



**解决办法：**

1.`清理磁盘`  /  `磁盘扩容`  /  `修改kubelet默认配置`

2.`重启对应节点kubelet`

```shell
$ systemctl restart kubelet
```

3.`重启pod`

```shell
$ kubectl delete pod node-exporter-qxvnw -n monitoring
```



## 5.k8s资源限制

```yaml
https://juejin.cn/post/6974203884369608734
requests：相对限制，是容器的最低申请资源，这个限制是相对的，无法做到绝对严格。
limits：绝对限制，这个是限制的绝对的，不可能超越。


resources:
      limits:
        cpu: "2"
        memory: 1000Mi
      requests:
        cpu: "1"
        memory: 500Mi
        
```



## 6.k8s node节点停机维护，pod如何迁移？

```shell
# 1. 默认迁移
当node节点关机后，k8s集群并没有立刻发生任何自动迁移动作，如果该node节点上的副本数为1，则会出现服务中断的情况。其实事实并非如此，k8s在等待5分钟后，会自动将停机node节点上的pod自动迁移到其他node节点上。

模拟node节点停机，停止kubelet
$ systemctl stop kubelet

$ kubectl get node
... NotReady

监控pod状态，大约5分钟
$ kubectl get pod -n test -o wide -n test -w

5分钟后，pod终止并重建
从以上过程看出，停机node节点上的pod在5分钟后先终止再重建，直到pod在新节点启动并由readiness探针检测正常后并处于1\1 Running状态才可以正式对外提供服务。因此服务中断时间=停机等待5分钟时间+重建时间+服务启动时间+readiness探针检测正常时间

kubelet停止后，node节点自动添加了污点（Taints）
$ kubectl describe node k8s-3-218

可以看到，此时pod的Tolerations 默认对于具有相应Taint的node节点容忍时间为300s

# 2. 手动迁移
为避免等待默认的5分钟，我们还可以使用cordon、drain、uncordor三个命令实现节点的主动维护。此时需要用到以下三个命令：

cordon：标记节点不可调度，后续新的pod不会被调度到此节点，但是该节点上的pod可以正常对外服务；
drain：驱逐节点上的pod至其他可调度节点；
uncordon：标记节点可调度；

a. 标记节点不可调度：
$ kubectl cordon k8s-3-219
$ kubectl get node

b. 驱逐pod
$ kubectl drain k8s-3-219 --delete-local-data --ignore-daemonsets --force

此时与默认迁移不同的是，pod会先重建再终止，此时的服务中断时间=重建时间+服务启动时间+readiness探针检测正常时间，必须等到1/1 Running服务才会正常

# 3. 如何能够做到平滑迁移呢
论是默认迁移和手动迁移，都会导致服务中断，而pdb能可以实现节点维护期间不低于一定数量的pod正常运行，从而保证服务的可用性
要做到平滑迁移就需要用的pdb(PodDisruptionBudget)，即主动驱逐保护

从218驱逐到219
1. 标记节点不可调度
2. 新建pdb

此时由于只有一个副本，最小可用为1，则allow就为0，因此在一个副本通过pdb是不会发生驱逐的，我们需要先扩容，将副本数调整为大于1.
调整副本数replicas为2
$ kubectl edit deploy helloworld -n test
再次查看pdb
$ kubectl get pdb
ALLOWED为1

3. 驱逐
4. 维护完毕，将node调整为可调度
$ kubectl uncordon k8s-3-218


```



## 7.k8s监控方案

https://cloud.tencent.com/developer/article/2047144

```shell
需要关注以下三个方面：

- Kubernetes集群本身的监控，主要是kubernetes的各个组件
- kubernetes集群中Pod的监控，Pod的CPU、内存、网络、磁盘等监控
- 集群内部应用的监控，针对应用本身的监控
```

```
在kubernetes中的监控需要考虑到这几个方面：

应该给Pod打上哪些label，这些label将成为监控的metrics。
当应用的Pod漂移了之后怎么办？因为要考虑到Pod的生命周期比虚拟机和物理机短的多，如何持续监控应用的状态？
更多的监控项，kubernetes本身、容器、应用等。
监控指标的来源，是通过heapster收集后汇聚还是直接从每台主机的docker上取？
```

```
通过Prometheus提供的服务自动发现机制，可以实现对Kubernetes集群的自动化监控：当在集群中部署了新的应用时，Prometheus可以自动发现Pod、Service等相关的监控信息；当相关的资源被从Kubernetes集群中删除时，Prometheus也会自动移除这个应用相关的资源监控。
```

https://help.aliyun.com/document_detail/123394.html

https://cloud.tencent.com/developer/article/2146317?from=article.detail.2146323&areaSource=106000.1&traceId=6NSXQa8eifmGWXUAOSTxn 上篇





https://www.cnblogs.com/cyh00001/p/16725312.html

### 1.监控Kubernetes集群

#### 1.安装Prometheus

```yaml
1.创建命名空间

$ vim namespace.yml
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
```

```yaml
2.创建RBAC规则
RBAC为Kubernetes的授权认证方式，该规则用于授权Prometheus获取资源信息

$ vim prometheus-rbac.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: monitoring
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources: ["nodes", "nodes/proxy", "services", "endpoints", "pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: monitoring
  
  
查看RBAC
$ kubectl  get sa prometheus -n monitoring
$ kubectl get ClusterRole prometheus 
$ kubectl get ClusterRoleBinding prometheus -n monitoring
```

```yaml
3.创建Configmap
为了让镜像 和 配置文件解耦，以便实现镜像的可移植性和可复用性。
我们使用Configmap来管理Prometheus的配置文件，此处先使用默认的配置，用于启动Prometheus，后面再根据需要进行修改

$ vim prometheus-config.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval:     15s 
      evaluation_interval: 15s
    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
        - targets: ['localhost:9090']
        
```

```yaml
4. 部署Deployment
部署Prometheus的实例，并通过Volume挂载的方式，将Prometheus的配置文件挂载到Pod内。另外，在正式环境中建议通过PVC的方式

$ vim prometheus-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
  labels:
    app: prometheus
spec:
  strategy:
    type: Recreate
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - image: prom/prometheus:v2.20.0
        name: prometheus
        command:
        - "/bin/prometheus"
        args:
        - "--config.file=/etc/prometheus/config/prometheus.yml"
        - "--storage.tsdb.path=/data"
        - "--web.enable-lifecycle"
        securityContext:
          runAsUser: 0
        ports:
        - containerPort: 9090
          protocol: TCP
        volumeMounts:
        - mountPath: "/etc/prometheus/config/"
          name: config
        - name: host-time
          mountPath: /etc/localtime
      serviceAccountName: prometheus
      volumes:
      - name: config
        configMap:
          name: prometheus-config
      - name: host-time
        hostPath:
          path: /etc/localtime
```

```yaml
5. 创建Service

$ vim prometheus-service.yml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: prometheus
  name: prometheus
  namespace: monitoring
spec:
  ports:
  - name: "web"
    port: 9090
    protocol: TCP
    targetPort: 9090
  selector:
    app: prometheus
  type: NodePort
```

访问prometheus：

```shell
$ kubectl get svc -n monitoring
NAME         TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
prometheus   NodePort   10.233.66.15   <none>        9090:36223/TCP   7h6m
```



查看prometheus服务为nodePort 36223端口：

http://192.168.254.51:36223

查看Targets目标，当前除了监控Prometheus自身实例，还未有其他Kubernetes资源



#### 2.Kubernetes的服务发现

```
1. node角色
2. service角色
3. Pod角色
4. endpoints角色
5. ingress角色
```

##### 1.监控Node节点

1.`Daemonset部署node-exporter`

```yaml
$ vim node_exporter-daemonset.yml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
  labels:
    app: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      tolerations:   # 污点容忍
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - image: prom/node-exporter
        name: node-exporter
        ports:
        - name: scrape
          containerPort: 9100
          hostPort: 9100
      hostNetwork: true
      hostPID: true
      securityContext:
        runAsUser: 0
        
        
```

2.`Prometheus配置任务`

```yaml
$ vim prometheus-config.yml
      - job_name: 'kubernetes-node'
        kubernetes_sd_configs:
        - role: node
        relabel_configs:
        - source_labels: [__address__]
          regex: '(.*):10250'
          replacement: '${1}:9100'
          target_label: __address__
          action: replace
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
```

然后，重启prometheus

```shell
$ kubectl get pod -n monitoring | grep prometheus
prometheus-7f8d9bb5cb-sd8ff   1/1     Running   0          164m

$ kubectl delete pod prometheus-7f8d9bb5cb-sd8ff -n monitoring
```

再次访问prometheus：

![image-20230318213438958](D:\Tech\linux\System\assets\image-20230318213438958.png)





##### 2.监控容器

prometheus-config.yml文件中添下如下任务：

```yaml
$ vim prometheus-config.yml
      - job_name: 'kubernetes-cadvisor'
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        kubernetes_sd_configs:
        - role: node
        relabel_configs:
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
          
```

应用配置文件，重启prometheus生效

##### 3.监控Kube API Server

```yaml
$ vim prometheus-config.yml
      - job_name: 'kubernetes-apiservers'
        kubernetes_sd_configs:
        - role: endpoints
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        relabel_configs:
        - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
          action: keep
          regex: default;kubernetes;https
        - target_label: __address__
          replacement: kubernetes.default.svc:443
          
```

##### 4.监控Kubelet组件

prometheus-config.yml文件中添下如下任务

```yaml
$ vim prometheus-config.yml
      - job_name: 'k8s-kubelet'
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        kubernetes_sd_configs:
        - role: node
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics
```

##### 5.监控Kubernetes资源

Kubernetes资源对象包括Pod、Deployment、StatefulSets等

使用开源的kube-state-metrics方案来获取监控指标。



**1. 部署kube-state-metrics**

kube-state-metrics对Kubernetes有版本要求，如下图。

![image-20230319140515803](D:\Tech\linux\System\assets\image-20230319140515803.png)



```shell
下载项目仓库
$ git clone https://github.com/kubernetes/kube-state-metrics.git
```

```shell
部署安装
$ cd kube-state-metrics/
$ kubectl  apply -f examples/standard/

$ kubectl  get deploy kube-state-metrics -n kube-system   # 查看服务
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
kube-state-metrics   1/1     1            1           6m20s
```

prometheus-config.yml文件中添下如下任务

```yaml
$ vim prometheus-config.yml
      - job_name: kube-state-metrics
        kubernetes_sd_configs:
        - role: endpoints
        relabel_configs:
        - source_labels: [__meta_kubernetes_service_label_app_kubernetes_io_name]
          regex: kube-state-metrics
          replacement: $1
          action: keep
        - source_labels: [__address__]
          regex: '(.*):8080'
          action: keep
```

##### 6.监控service访问

在Kubernetes集群中，我们可以采用黑盒监控的模式，由Prometheus通过探针的方式对service进行访问探测，以便及时了解业务的可用性。

要实现探针检测，我们需要在集群中安装Blackbox Exporter。



1.`部署Blackbox Exporter`

```yaml
$ vim blackbox-exporter.yml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: blackbox-exporter
  name: blackbox-exporter
  namespace: monitoring
spec:
  ports:
  - name: blackbox
    port: 9115
    protocol: TCP
  selector:
    app: blackbox-exporter
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: blackbox-exporter
  name: blackbox-exporter
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: blackbox-exporter
  template:
    metadata:
      labels:
        app: blackbox-exporter
    spec:
      containers:
      - name: blackbox-exporter
        image: prom/blackbox-exporter
        imagePullPolicy: IfNotPresent
```

2.`Prometheus配置任务`

```yaml
$ vim prometheus-config.yml
      - job_name: 'kubernetes-services'
        kubernetes_sd_configs:
        - role: service
        metrics_path: /probe
        params:
          module: [http_2xx]
        relabel_configs:
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_probe]  
          action: keep
          regex: true
        - source_labels: [__address__]
          target_label: __param_target
        - target_label: __address__
          replacement: blackbox-exporter.monitoring.svc.cluster.local:9115
        - source_labels: [__param_target]
          target_label: instance
        - action: labelmap
          regex: __meta_kubernetes_service_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_service_name]
          target_label: kubernetes_name
```

```
该任务通过service角色发现的方式，获取集群中的service对象；并使用“prometheus.io/probe: true”标签进行过滤，只有包含此注解的service才纳入监控；另外，__address__执行Blackbox Exporter实例的访问地址，并且重写了标签instance的内容。
```

#### 3.Grafana展示

https://juejin.cn/post/7145097927067697159

#### 4.配置告警规则

https://blog.csdn.net/yanggd1987/article/details/109357238

https://blog.51cto.com/u_15060465/4244148

---



### 2.监控外部 Kubernetes 集群

https://ost.51cto.com/posts/12449

```
实际环境中很多企业是将 Prometheus 单独部署在集群外部的。所以监控外部集群非常有必要。

- 首先构造 Prometheus 连接 APIServer 的信息，在通过 kubernetes_sd_configs 做服务发现的时候只需要填入 Kubernetes 集群的 api_server、ca_file、bearer_token_file 信息即可。
```

创建用于 Prometheus 访问 Kubernetes 资源对象的 RBAC 对象:

```yaml
$ vim prometheus-rbac.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: monitoring

---
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: monitoring-token
  namespace: monitoring
  annotations:
    kubernetes.io/service-account.name: "prometheus"

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources: ["nodes", "nodes/proxy", "services", "endpoints", "pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: monitoring
```

```shell
$ kubectl apply -f prometheus-rbac.yml
```

获取上面的 Prometheus 对应的 Secret 的信息:

```shell
$ kubectl get secret -n monitoring
NAME               TYPE                                  DATA   AGE
monitoring-token   kubernetes.io/service-account-token   3      24s
```

```shell
$ kubectl describe secret monitoring-token -n monitoring
Name:         monitoring-token
Namespace:    monitoring
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: prometheus
              kubernetes.io/service-account.uid: f9000dcf-d529-48af-b283-1fa69194426f

Type:  kubernetes.io/service-account-token

Data
====
namespace:  10 bytes
token:      <token string>
ca.crt:     1099 bytes
```

上面的 token 和 ca.crt 信息就是我们用于访问 APIServer 的数据，可以将 token 信息保存到一个名为 k8s.token 的文本文件中



修改prometheus配置文件：

```yaml
$ vim prometheus.yml
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]


  - job_name: k8s-cadvisor
    honor_timestamps: true
    metrics_path: /metrics
    scheme: https
    kubernetes_sd_configs:  # kubernetes 自动发现
    - api_server: https://192.168.254.51:6443  # apiserver 地址
      role: node  # node 类型的自动发现
      bearer_token_file: k8s.token
      tls_config:
        insecure_skip_verify: true
    bearer_token_file: k8s.token
    tls_config:
      insecure_skip_verify: true
    relabel_configs:
    - action: labelmap
      regex: __meta_kubernetes_node_label_(.+)
    - separator: ;
      regex: (.*)
      target_label: __address__
      replacement: 192.168.254.51:6443
      action: replace
    - source_labels: [__meta_kubernetes_node_name]
      separator: ;
      regex: (.+)
      target_label: __metrics_path__
      replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor
      action: replace
```

现在去 Prometheus 页面就可以看到采集的外部 Kubernetes 集群的数据了

![image-20230318182631495](D:\Tech\linux\System\assets\image-20230318182631495.png)



## 8.镜像下载策略

主要分为三种：

- Always：总是从指定的仓库中获取镜像。
- Never：使用本地镜像，不从仓库中下载。
- IfNotPresent：当本地镜像不存在时，才从仓库拉取。



## 9.持久化

1）EmptyDir（空目录）：没有指定要挂载宿主机上的某个目录，直接由 Pod 内保部映射到宿主机上。类似于 docker 中的 manager volume。

2）Hostpath：将宿主机上已存在的目录或文件挂载到容器内部。类似于 docker 中的 bind mount 挂载方式。

3）PersistentVolume（简称 PV）： 基于 NFS 服务的 PV，也可以基于 GFS 的 PV。它的作用是统一数据持久化目录，方便管理。



### 1.Hostpath方式持久化：

```yaml
# 将主机路径挂载到pod里面

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: grafana
  name: grafana
spec:
  replicas: 3
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - image: grafana/grafana-enterprise:8.2.0
        name: grafana
        volumeMounts:   # 需要挂载的pod路径
        - name: grafana-data
          mountPath: /var/lib/grafana

        - name: grafana-config
          mountPath: /etc/grafana

      volumes:
      - name: grafana-data  
        hostPath:
          path: /data/grafana/data  # 服务器节点路径

      - name: grafana-config 
        hostPath:
          path: /data/grafana/etc  # 服务器节点路径
```

### 2.pv & pvc存储

#### 1.概念

```shell
# 概念

PersistentVolume  (简称PV):  由管理员设置的存储
PersistentVolumeClaim (简称PVC): 是用户存储的请求

- 关于PersistentVolume的访问方式:
ReadWriteOnce - 卷以读写方式挂载到单个节点
ReadOnlyMany  - 卷以只读方式挂载到多个节点
ReadWriteMany - 卷以读写方式挂载到多个节点

- 关于PersistentVolume (PV) 状态:
Available(可用状态)   -   一块空闲资源还没有被任何声明绑定
Bound(绑定状态)       -   声明分配到PVC进行绑定，PV进入绑定状态
Released(释放状态)    -   PVC被删除，PV进入释放状态，等待回收处理
Failed(失败状态)      -   PV执行自动清理回收策略失败

- 关于PersistentVolumeClaims (PVC) 状态:
Pending(等待状态)     -   等待绑定PV
Bound(绑定状态)       -   PV已绑定PVC
```

```shell
# StorageClass
- PV是运维人员来创建的，开发操作PVC，可是大规模集群中可能会有很多PV，如果这些PV都需要运维手动来处理这也是一件很繁琐的事情，所以就有了动态供给概念，也就是Dynamic Provisioning。

- 而我们上面的创建的PV都是静态供给方式，也就是Static Provisioning。而动态供给的关键就是StorageClass，它的作用就是创建PV模板。

- 创建StorageClass里面需要定义PV属性比如存储类型、大小等；另外创建这种PV需要用到存储插件。最终效果是，用户提交PVC，里面指定存储类型，如果符合我们定义的StorageClass，则会为其自动创建PV并进行绑定。


```

#### 2.本地持久化存储

本地持久化存储（Local Persistent Volume）就是把数据存储在POD运行的宿主机上

```shell
# 为什么需要这种类型的存储呢？
- 有时候你的应用对磁盘IO有很高的要求，网络存储性能肯定不如本地的高，尤其是本地使用了SSD这种磁盘。
```

```shell
- 但这里有个问题，通常我们先创建PV，然后创建PVC，这时候如果两者匹配那么系统会自动进行绑定，
- 哪怕是动态PV创建，也是先调度POD到任意一个节点，然后根据PVC来进行创建PV然后进行绑定最后挂载到POD中，
- 可是本地持久化存储有一个问题就是这种PV必须要先准备好，而且不一定集群所有节点都有这种PV，如果POD随意调度肯定不行，

- 如何保证POD一定会被调度到有PV的节点上呢？
这时候就需要在PV中声明节点亲和，且POD被调度的时候还要考虑卷的分布情况。
```

```yaml
# 创建pv
$ cat grafana-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:     # PV建立不要加名称空间，因为PV属于集群级别的
  name: grafana-data-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: grafana-data-storage
  local:
    path: /data/grafana/data       # PV对应的本地磁盘的路径
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - k8s-master      # 指定节点名称，磁盘存在与k8s-master节点上，也就意味着 Pod使用这个 PV就必须运行在 k8s-master 上

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: grafana-config-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: grafana-config-storage
  local:
    path: /data/grafana/etc
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - k8s-master
```



```yaml
# 创建pvc
$ cat grafana-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-data-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: grafana-data-storage
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-config-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: grafana-config-storage
```



```yaml
# 创建deployment
$ cat grafana-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: grafana
  name: grafana
spec:
  replicas: 3
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - image: grafana/grafana-enterprise:8.2.0
        name: grafana
        volumeMounts:   # 需要挂载的pod路径
        - name: grafana-data
          mountPath: /var/lib/grafana

        - name: grafana-config
          mountPath: /etc/grafana

      volumes:
      - name: grafana-data  
        persistentVolumeClaim:
          claimName: grafana-data-claim   # 服务器节点路径

      - name: grafana-config 
        persistentVolumeClaim:
          claimName: grafana-config-claim  # 服务器节点路径
```



```yaml
# 创建服务
$ cat garana-svc.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: grafana
  name: grafana
spec:
  ports:
  - port: 3000
    protocol: TCP
    targetPort: 3000
  selector:
    app: grafana
  type: NodePort
```

------

**延迟绑定**

```yaml
# 定义pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local: # local类型
    path: /data/vol1  # 节点上的具体路径
  nodeAffinity: # 这里就设置了节点亲和
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node01 # 这里我们使用node01节点，该节点有/data/vol1路径
```

```yaml
# 定义存储类
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer    # 延迟绑定
```

```yaml
# 定义pvc
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: local-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: local-storage
```

```yaml
# 定义pod
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      appname: myapp
  template:
    metadata:
      name: myapp
      labels:
        appname: myapp
    spec:
      containers:
      - name: myapp
        image: tomcat:8.5.38-jre8
        ports:
        - name: http
          containerPort: 8080
          protocol: TCP
        volumeMounts:
          - name: tomcatedata
            mountPath : "/data"
      volumes:
        - name: tomcatedata
          persistentVolumeClaim:
            claimName: local-claim
```

