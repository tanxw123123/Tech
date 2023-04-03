## 1.安装

```shell
环境：ubuntu 20.04

$ sudo apt update
$ sudo apt upgrade
$ sudo apt install ansible
$ ansible --version
```

```shell
$ ssh-keygen     # 创建秘钥对

- 然后将公钥拷贝到远程主机 authorized_keys
```

## 2.创建项目

```shell
$ mkdir project01 && cd project01

$ cp /etc/ansible/ansible.cfg ./       # 拷贝ansible配置文件到项目目录
[defaults]
#inventory      = inventory/
host_key_checking = False


$ mkdir inventory                # 创建存放hosts文件目录
$ cd /home/ubuntu/project01/inventory
$ vim xxx.hosts          # 创建主机列表
[xxx-admin]
xxx-test-jp-admin-1
xxx-test-jp-admin-2
xxx-test-jp-admin-3

[wsdy-batch]
xxx-test-jp-batch-v23-1
xxx-test-jp-batch-v23-2

```

## 3.配置解析

```shell
$ cd ~/.ssh
$ vim config
Host *
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null
    IgnoreUnknown UseKeychain
    UseKeychain yes
    AddKeysToAgent yes
    ForwardAgent yes
    ServerAliveInterval  60
    ServerAliveCountMax  60
    Protocol 2
    AddKeysToAgent yes

#####xxx-test####
Host xxx-test-jp-admin-1
    user ubuntu
    hostname 35.xx.xx.172
    port 22
Host xxx-test-jp-admin-2
    user ubuntu
    hostname 18.xx.xx.227
    port 22
Host xxx-test-jp-admin-3
    user ubuntu
    hostname 35.xx.xx.206
    port 22
Host xxx-test-jp-batch-v23-1
    user ubuntu
    hostname 13.xx.xx.18
    port 22
Host xxx-test-jp-batch-v23-2
    user ubuntu
    hostname 52.xx.xx.125
    port 22
```

## 4.基础命令

```shell
$ cd /home/unbuntu/project01
$ ansible all -i inventory/xxx.hosts --list-hosts      # 查看主机列表
$ ansible all -i inventory/xxx.hosts -m ping           # 测试主机连通性
$ ssh xxx-test-jp-admin-1                          # 配置了本地解析，所以可以直接ssh连接主机名
```

## 5.role角色

```shell
$ cd ~/project01 && mkdir roles
$ cd roles
$ ansible-galaxy init ossec     # 创建角色及目录结构
$ cd ossec/tasks
```

```yaml
$ vim main.yml     # 角色主任务
---
# tasks file for ossec
- name: test
  shell: touch /root/100.txt

main.yml 文件也可以写成include的形式包含多个yml

$ vim main.yml
- include_tasks: _main.yml
- include_tasks: filebeat.yml
```

```yaml
# 应用角色
$ cd ~/project01
$ vim deploy.yml
- name: hostname,init
  hosts: all
  gather_facts: no
  become: yes             # 远程主机提权root执行
  roles:
    - { role: hostname, tags: hostname }
    - { role: init, tags: init }

- name: ossec
  hosts: all
  gather_facts: no
  become: yes
  roles:
    - ossec
  tags:
    - ossec
```

```shell
$ ansible-playbook -i inventory/xxx.hosts deploy.yml      # 执行playbook

$ ansible-playbook -i inventory/xxx.hosts deploy.yml -t hostname,init -vv     

-t 参数指定tag执行
```



## 6.playbook

### 1.jinja2模板

```jinja2
$ vim templates/config.j2
USER_AGENT_SERVER_IP="{{ server_ip }}"        # 

- 定义变量
$ vim vars/main.yml
server_ip: 10.10.10.10

$ vim tasks/install_ossec.yml
- name: ossec config
  template: src=config.j2 dest=/var/ossec/etc/ossec.conf mode=0640
```



### 2.变量

1. 将变量放在剧本开头的vars中

```yaml
- name: hostname,init
  hosts: all
  gather_facts: no
  become: yes
  vars:
    server_ip: 10.10.10.10
```



2.外部文件定义变量

```yaml
$ cat vars/users.yml
user: tom
shell: /sbin/nologin


$ vim deploy.yaml
- name: hostname,init
  hosts: all
  gather_facts: no
  become: yes
  vars_files:
    - vars/users.yml
```



### 3.loop循环

创建用户

```yaml
- name: create user
  user:
    name:  "{{ item }}"
    shell: /sbin/nologin
    state: present
  loop:
    - tom
    - lucy
```

```yaml
- 使用变量

$ vim vars/main.yaml
user_name:
  - tom
  - lucy
  

$ vim  xxx/create_user.yaml
- name: create user
  user:
    name:  "{{ item }}"
    shell: /sbin/nologin
    state: present
  loop: "{{ user_name }}"

```

