```
rsync实现细节：
- 源文件、目标文件。以哪边文件为同步基准
- --delete  删除源主机没有，目标主机有的文件，以源主机为基准
- 遇到软链接时是拷贝软链接本身还是拷贝软链接所指向的文件
```
```
rsync的三种工作方式：
1. 本地文件系统上同步
格式：
Local:  rsync [OPTION...] SRC... [DEST]

2. 本地主机使用远程shell和远程主机通信
格式：
Access via remote shell:
  Pull: rsync [OPTION...] [USER@]HOST:SRC... [DEST]
  Push: rsync [OPTION...] SRC... [USER@]HOST:DEST

3. 本地主机通过网络套接字连接远程主机上的rsync daemon
格式：
Access via rsync daemon:
  Pull: rsync [OPTION...] [USER@]HOST::SRC... [DEST]
        rsync [OPTION...] rsync://[USER@]HOST[:PORT]/SRC... [DEST]
  Push: rsync [OPTION...] SRC... [USER@]HOST::DEST
        rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/DEST

. 前两者的本质是通过管道通信，即使是远程shell。而方式(3)则是让远程主机上运行rsync服务，使其监听在一个端口上，等待客户端的连接
. user@host:path或user@host::path
. 如果主机和path路径之间使用单个冒号隔开，表示使用的是远程shell通信方式，而使用双冒号隔开的则表示的是连接rsync daemon
```
```
源路径如果是一个目录的话，带上尾随斜线和不带尾随斜线是不一样的，不带尾随斜线表示的是整个目录包括目录本身，带上尾随斜线表示的是目录中的文件，不包括目录本身，如：
# rsync -av /etc /tmp
# rsync -av /etc/ /tmp
第一个命令会在/tmp目录下创建etc目录，而第二个命令不会在/tmp目录下创建etc目录，源路径/etc/中的所有文件都直接放在/tmp目录下。
```
```
选项说明：
-v：显示rsync过程中详细信息。可以使用"-vvvv"获取更详细信息。
-a --archive  ：归档模式，表示递归传输并保持文件属性
-p --perms：保持perms属性(权限，不包括特殊权限)。
-z        ：传输时进行压缩提高效率。
```
```
rsync daemon模式介绍：
  既然rsync通过远程shell就能实现两端主机上的文件同步，还要使用rsync的服务干啥？试想下，你有的机器上有一堆文件需要时不时地同步到众多
机器上去，比如目录a、b、c是专门传输到web服务器上的，d/e、f、g/h是专门传输到ftp服务器上的，还要对这些目录中的某些文件进行排除，如果
通过远程shell连接方式，无论是使用排除规则还是包含规则，甚至一条一条rsync命令地传输，这都没问题，但太过繁琐且每次都要输入同样的命令
显得太死板。使用rsync daemon就可以解决这种死板问题。

实例：
服务端配置
1. 修改配置文件
# vim /etc/rsyncd.conf
uid = root
gid = root
use chroot = no
max connections = 200
timeout = 300
port = 51800

[data]
path = /data/
ignore errors
read only = false
list = false
auth users = admin
secrets file = /etc/rsyncd.secrets
hosts allow = 0.0.0.0/0
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log

2. 创建保存auth user用户列表的用户名密码文件 /etc/rsync.password
# cat /etc/rsyncd.secrets
admin:123456

3. 启动rsync daemon
ubuntu系统
# rsync --daemon

centos系统
# systemctl start rsyncd

---------------------------------------------------
客户端配置
# cat /etc/rsync.pass
123456

测试：
# rsync -avrp --delete --port=51800 source1/ rsync@192.168.254.12::data --password-file=/etc/rsync.pass
```
