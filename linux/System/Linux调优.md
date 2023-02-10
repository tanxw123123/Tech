## 1.系统初始化

ubuntu:

```shell
###################### 虚拟机初始化 ###################

# Ubuntu 20.04 修改网卡名称为eth0
$ vim /etc/default/grub
GRUB_CMDLINE_LINUX=""
修改为：
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"

$ update-grub
$ vim /etc/netplan/00-installer-config.yaml
将网卡名修改为 eth0
$ reboot
------------------------------------------------------
# Ubuntu20.04 设置静态IP
vim /etc/netplan/00-installer-config.yaml

network:
  ethernets:
    eth0:
      dhcp4: no
      addresses:
      - 192.168.254.70/24
      gateway4: 192.168.254.254
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
  version: 2
$ netplan apply    # 启用配置
------------------------------------------------------
# 启用ssh
$ apt update
$ apt install openssh-server
$ systemctl status ssh

# 创建普通用户
$ useradd  -d  /home/ubuntu   -m -s  /bin/bash  ubuntu
$ usermod -G sudo ubuntu    # 添加sudo权限,加入sudo组，可以不用配置这一步，直接修改/etc/sudoers
$ echo "ubuntu:xxx" | chpasswd    # 设置ubuntu用户的密码为xxx

# 开启sudo免密登录
$ vim /etc/sudoers
ubuntu  ALL=(ALL:ALL) NOPASSWD:ALL
# !!!注意： 有的时候你的将用户设了nopasswd，但是不起作用，原因是被后面的group的设置覆盖了，需要把group的设置也改为nopasswd。

# 需要修改时间为24小时，可以修改/etc/default/locale，默认没有LC_TIME这个变量，在文件中增加一行
$ vim /etc/default/locale
LC_TIME=en_DK.UTF-8
------------------------------------------------------
# 禁用ipv6
$ vim /etc/sysctl.conf
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
net.ipv6.conf.lo.disable_ipv6=1

$ sysctl -p
```

```shell
---
- name: init apt
  shell: |
    echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections
    pkill apt
    pkill dpkg
    rm -rf /var/lib/dpkg/lock
    rm -rf /var/lib/dpkg/lock-frontend
    rm -rf /var/cache/debconf
    dpkg --configure -a --force-confold --force-confdef
    apt --fix-broken install -y
    apt update
    apt -o Dpkg::Options::=--force-confold -o Dpkg::Options::=--force-confdef upgrade -y

- name: locales and timezone ...
  shell: |
    # 编码
    apt install locales -y
    locale-gen en_US.UTF-8
    locale-gen zh_CN.UTF-8
    # 时区
    apt install tzdata chrony -y
    cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
    echo 'Asia/Shanghai' > /etc/timezone
    timedatectl set-timezone Asia/Shanghai

- name: install necessary software  ...
  shell: |
    apt install -y \
    gcc g++ make net-tools unzip fish git vim openjdk-8-jdk

- shell: |
    cat>/etc/security/limits.conf<<EOF
    root  soft  nofile    65536
    root  hard  nofile    65536
    root  soft  nproc     65536
    root  hard  nproc     65536
    root  soft  nofile    65536
    root  soft  memlock   unlimited
    root  hard  memlock   unlimited
    *  soft  nofile    65536
    *  hard  nofile    65536
    *  soft  nproc     65536
    *  hard  nproc     65536
    *  soft  nofile    65536
    *  soft  memlock   unlimited
    *  hard  memlock   unlimited
    EOF

- name: apt update ; upgrade ; autoremove   ...
  shell: |
    apt update ; apt upgrade -y ; apt autoremove -y
```



centos

```shell
# timezone ...
yum install epel-release tzdata chrony -y
cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
echo 'Asia/Shanghai' > /etc/timezone
timedatectl set-timezone Asia/Shanghai
date -R

#install necessary software  ...
yum install -y \
gcc g++ make net-tools unzip fish git vim wget
 
#
cat>/etc/security/limits.conf<<EOF
root  soft  nofile    65536
root  hard  nofile    65536
root  soft  nproc     65536
root  hard  nproc     65536
root  soft  nofile    65536
root  soft  memlock   unlimited
root  hard  memlock   unlimited
*  soft  nofile    65536
*  hard  nofile    65536
*  soft  nproc     65536
*  hard  nproc     65536
*  soft  nofile    65536
*  soft  memlock   unlimited
*  hard  memlock   unlimited
EOF
```

---



