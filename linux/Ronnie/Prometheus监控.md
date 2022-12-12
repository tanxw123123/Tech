## 1.简介

​        Prometheus 是由 SoundCloud 开发的开源监控报警系统和时序列数据库。Prometheus 的基本原理是通过 HTTP 周期性抓取被监控组件的状态，任意组件只要提供对应的 HTTP 接口并且符合 Prometheus 定义的数据格式，就可以接入 Prometheus 监控。

​        Prometheus Server 负责定时在目标上抓取 metrics（指标）数据并保存到本地存储里面。Prometheus 采用了一种 Pull（拉）的方式获取数据，不仅降低客户端的复杂度，客户端只需要采集数据，无需了解服务端情况，而且服务端可以更加方便的水平扩展。

**a. 原理图：**

![image-20220903170837259](D:\Tech\linux\Ronnie\.assets\image-20220903170837259.png)

**b. 相关组件：**

- Prometheus 服务器
  Prometheus Server 是 Prometheus组件中的核心部分，负责实现对监控数据的获取，存储以及查询。
- NodeExporter 业务数据源
  业务数据源通过 Pull/Push 两种方式推送数据到 Prometheus Server。
- AlertManager 报警管理器
  Prometheus 通过配置报警规则，如果符合报警规则，那么就将报警推送到 AlertManager，由其进行报警处理。
- 可视化监控界面
  Prometheus 收集到数据之后，由 WebUI 界面进行可视化图标展示。目前我们可以通过自定义的 API 客户端进行调用数据展示，也可以直接使用 Grafana 解决方案来展示。

**c. 简单介绍一下prometheus的pull和push模式：**

- pull模式：客户端使用library，变成exporter，然后prometheus server定时从exporter pull数据。

- push模式：使用pushgateway，所有客户端push数据到pushgateway，然后prometheus server定时从pushgateway pull数据。

比较：

push模式的缺点：采用pushgateway的方式，如果某一个上报方挂了，pushgateway无法感知上报方的状态，所以这时候如果不做任何操作，那么prometheus依旧会从pushgateway中获取到旧的、不正确的数据。

pull的优点：采用exporter的方式，如果某一个exporter挂掉了，那么prometheus就pull不到数据，那么时序数据库没有新的数据产生。这是正确的。

所以pull模式是prometheus的推荐模式。

| 方式     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| Pull方式 | 服务端主动向客户端拉取数据，需要客户端上安装exporters(导出器)作为守护进程。官网提供大量exporters下载，比如使用最多的node_exporters，它几乎可以采集全部系统数据，默认监听9100端口。 |
| Push方式 | 客户端安装pushgateway插件，而且需要运维人员用脚本把监控数据组织成键值形式提交给pushgateway，再由它提交给服务端。它适合于当exporters无法满足需求时的定制化监控。 |

**d. 发现：**

在Prometheus的配置中，一个最重要的概念就是数据源target，而数据源的配置主要分为静态配置和动态发现，大致为以下几类：

| 发现类型              | 说明                |
| --------------------- | ------------------- |
| static_configs        | 静态服务发现        |
| dns_sd_configs        | DNS 服务发现        |
| file_sd_configs       | 文件服务发现        |
| consul_sd_configs     | Consul 服务发现     |
| serverset_sd_configs  | Serverset 服务发现  |
| nerve_sd_configs      | Nerve 服务发现      |
| marathon_sd_configs   | Marathon 服务发现   |
| kubernetes_sd_configs | Kubernetes 服务发现 |
| gce_sd_configs        | GCE 服务发现        |
| ec2_sd_configs        | EC2 服务发现        |
| openstack_sd_configs  | OpenStack 服务发现  |
| azure_sd_configs      | Azure 服务发现      |
| triton_sd_configs     | Triton 服务发现     |

---



## 2.服务端安装

https://prometheus.io/download   # 下载安装包

安装：

```shell
$ useradd prometheus
$ usermod -s /sbin/nologin prometheus
$ wget https://github.com/prometheus/prometheus/releases/download/v2.36.1/prometheus-2.36.1.linux-amd64.tar.gz -P /data
$ cd /data &&  tar -xf prometheus-2.36.1.linux-amd64.tar.gz
$ mv prometheus-2.36.1.linux-amd64 prometheus
$ chown -R prometheus.prometheus prometheus
```

配置systemctl启动：

```shell
$ cat /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/
After=network.target

[Service]
Type=simple
User=prometheus
ExecStart=/data/prometheus/prometheus \
    --config.file=/data/prometheus/prometheus.yml \
    --storage.tsdb.path=/data/prometheus/data \
    --storage.tsdb.retention=10d \
    --web.enable-admin-api \
    --web.enable-lifecycle
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

启动服务：

```shell
$ systemctl start prometheus
$ systemctl enable prometheus
```

---



## 3.安装NodeExporter

安装：

```shell
$ useradd prometheus
$ usermod -s /sbin/nologin prometheus 
$ wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz -P /data
$ cd /data && tar -xf node_exporter-1.3.1.linux-amd64.tar.gz
$ mv node_exporter-1.3.1 node_exporter
$ chown -R prometheus.prometheus node_exporter
```

配置systemctl启动：

```shell
$ cat /etc/systemd/system/node_exporter.service
[Service]
User=prometheus
Group=prometheus
ExecStart=/usr/local/node_exporter/node_exporter

[Install]
WantedBy=multi-user.target

[Unit]
Description=node_exporter
After=network.target
```

启动服务：

```shell
$ systemctl start node_exporter
$ systemctl enable node_exporter
```

---



## 4.安装alertmanager

我这里就安装在服务端：

```shell
$ cd /data
$ wget https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz
$ tar -xf alertmanager-0.24.0.linux-amd64.tar.gz
$ mv alertmanager-0.24.0.linux-amd64.tar.gz alertmanager
$ chown -R prometheus.prometheus alertmanager
```

配置systemctl启动：

```shell
$ cat /etc/systemd/system/alertmanager.service
[Unit]
Description=alertmanager
After=network.target

[Service]
User=prometheus
Type=simple
ExecStart=/data/alertmanager/alertmanager --config.file=/data/alertmanager/alertmanager.yml --storage.path=/data/alertmanager/data
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

启动服务：

```shell
$ systemctl start alertmanager
$ systemctl enable alertmanager
$ ss -tnlp|grep 9093
```

配置webhook告警规则：

```shell
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'web.hook'
receivers:
  - name: 'web.hook'
    webhook_configs:
      - url: 'http://10.0.27.224:8000/api/alert/prod'   # 通过自己编写的python程序告警
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

---



## 5.webhook告警程序

使用webhook方式，alertmanager会给配置的webhook地址发送一个http类型的post请求，参数为json字符串（字符串类型），如下（此处格式化为json了）：

```json
{
    "receiver":"webhook",
    "status":"resolved",
    "alerts":[
        {
            "status":"resolved",
            "labels":{
                "alertname":"hostCpuUsageAlert",
                "instance":"192.168.199.24:9100",
                "severity":"page"
            },
            "annotations":{
                "description":"192.168.199.24:9100 CPU 使用率超过 85% (当前值为: 0.9973333333333395)",
                "summary":"机器 192.168.199.24:9100 CPU 使用率过高"
            },
            "startsAt":"2020-02-29T19:45:21.799548092+08:00",
            "endsAt":"2020-02-29T19:49:21.799548092+08:00",
            "generatorURL":"http://localhost.localdomain:9090/graph?g0.expr=sum+by%28instance%29+%28avg+without%28cpu%29+%28irate%28node_cpu_seconds_total%7Bmode%21%3D%22idle%22%7D%5B5m%5D%29%29%29+%3E+0.85&g0.tab=1",
            "fingerprint":"368e9616d542ab48"
        }
    ],
    "groupLabels":{
        "alertname":"hostCpuUsageAlert"
    },
    "commonLabels":{
        "alertname":"hostCpuUsageAlert",
        "instance":"192.168.199.24:9100",
        "severity":"page"
    },
    "commonAnnotations":{
        "description":"192.168.199.24:9100 CPU 使用率超过 85% (当前值为: 0.9973333333333395)",
        "summary":"机器 192.168.199.24:9100 CPU 使用率过高"
    },
    "externalURL":"http://localhost.localdomain:9093",
    "version":"4",
    "groupKey":"{}:{alertname="hostCpuUsageAlert"}"
}
```

此时需要使用java（其他任何语言都可以，反正只要能处理http的请求就行）搭建个http的请求处理器来处理报警通知，我这里用python程序：

- 安装python相关模块：

```shell
$ pip3 install python-dateutil
$ pip3 install requests
$ pip3 install flask
```

- python代码如下：

```python
#!/usr/bin/python3
import datetime
from dateutil import parser
import requests
import json
from flask import Flask
from flask import request
app = Flask(__name__)

@app.route("/api/alert/haha", methods=["POST"])
def alarm():
    messages = request.get_data().decode('utf-8')
    messages = eval(messages)["alerts"]
    print(messages)

    for i in range(len(messages)):
        message = messages[i]
        status = "告警" if message["status"] == "firing" else "恢复"
        severity = message["labels"]["severity"]
        alertname = message["labels"]["alertname"]
        description = message["annotations"]["description"]

       # data = {
       #     "text": "状态:{0}\n{1}\n告警描述:{2}\n{3}\n".format(status, severity, alertname, description)
       #         }
        message = str('状态: %s' %status + '\n'
                '%s' %severity + '\n'
                '告警描述: %s' %alertname + '\n'
                '%s' %description + '\n'
        )
        sendtelegram(message)
    return "ok"

def sendtelegram(messages):
    url = "https://api.telegram.org/botToken值/sendMessage?chat_id=群组id&text=" + str(messages)
    requests.get(url)

if __name__ == '__main__':
    app.run('0.0.0.0', '8000', debug=True)
```

配置systemctl启动python程序：

```shell
$ cat /etc/systemd/system/python3.service
[Unit]
Description=Python3
Documentation=https://prometheus.io/
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/bin/python3 /script/alert-telegram.py run      # python代码路径自定义
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

启动python程序：

```shell
$ systemctl start python3
$ systemctl enable python3
```

测试curl发送POST请求：

```shell
$ curl -d "hello python3" -X POST http://ip:8000/api/alert/haha
```

---



## 6.Prometheus黑盒监控

在监控体系里面，通常我们认为监控分为：白盒监控和黑盒监控。

![image-20220905215408679](D:\Tech\linux\Ronnie\.assets\image-20220905215408679.png)

- 黑盒监控：主要关注的现象，一般都是正在发生的东西，例如出现一个告警，业务接口不正常，那么这种监控就是站在用户的角度能看到的监控，重点在于能对正在发生的故障进行告警。
- 白盒监控：主要关注的是原因，也就是系统内部暴露的一些指标，例如 redis 的 info 中显示 redis slave down，这个就是 redis info 显示的一个内部的指标，重点在于原因，可能是在黑盒监控中看到 redis down，而查看内部信息的时候，显示 redis port is refused connection。

​      **Blackbox Exporter 是 Prometheus 社区提供的官方黑盒监控解决方案，其允许用户通过：HTTP、HTTPS、DNS、TCP 以及 ICMP 的方式对网络进行探测**

1.安装blackbox_exporter

```shell
$ wget -P /data https://github.com/prometheus/blackbox_exporter/releases/download/v0.21.0/blackbox_exporter-0.21.0.linux-amd64.tar.gz
$ tar -xf blackbox_exporter-0.21.0.linux-amd64.tar.gz
$ mv blackbox_exporter-0.21.0.linux-amd64.tar.gz blackbox_exporter
$ chown -R prometheus.prometheus blackbox_exporter
```

配置systemctl启动：

```shell
$ cat /etc/systemd/system/blackbox_exporter.service
[Unit]
Description=blackbox_exporter
After=network.target

[Service]
User=prometheus
Type=simple
ExecStart=/data/blackbox_exporter/blackbox_exporter --config.file=/data/blackbox_exporter/blackbox.yml
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

启动服务：

```shell
$ systemctl start blackbox_exporter
$ systemctl enable blackbox_exporter
$ netstat -anptu|grep 9115
```

2.prometheus添加相关监控：

```shell
$ vim /data/prometheus/prometheus.yml
scrape_configs:
  - job_name: "blackbox"
    metrics_path: /probe
    params:
      module: [http_2xx]
    file_sd_configs:
    - refresh_interval: 1m
      files:
      - "/data/prometheus/blackbox/test.yml"
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115  # The blackbox exporter's real hostname:port.
```

```shell
$ chown -R prometheus.prometheus blackbox
```

---



## 7.配置告警规则

```shell
$ cd /data/prometheus && mkdir rules
$ vim /data/prometheus/rules/rules.yaml
groups:
- name: 证书还有30天过期
  rules:
  - alert: ssl证书过期告警
    expr: floor((probe_ssl_earliest_cert_expiry - time()) / 86400 < 200)
    for: 15s
    labels:
      severity: "告警等级: warning"
    annotations:
      summary: "告警主机: {{ $labels.job }}"
      description: '告警问题:{{ $labels.instance }}的证书还有{{ printf "%.1f" $value }}天就过期了，请尽快更新证书'
```

