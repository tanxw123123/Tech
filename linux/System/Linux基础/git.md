```shell
$ git push origin --delete dev         # 删除远程分支
$ git branch -d dev                    # 删除当前分支
$ git reset --hard HEAD                # 回滚代码
$ git checkout -b dev                  # 创建并切换分支
```

## 1.gitlab部署

```shell
$ sudo docker run -itd \
    -p 880:80 \
	-p 10022:22 \
	-v /data/gitlab/config:/etc/gitlab \
    -v /data/gitlab/logs:/var/log/gitlab \
    -v /data/gitlab/data:/var/opt/gitlab \
    --restart always \
    --name gitlab \
    gitlab/gitlab-ce:latest
```

修改配置

```shell
$ vim /data/gitlab/config/gitlab.rb
external_url 'http://192.168.254.199'             # gitlab访问地址，可以写域名
gitlab_rails['gitlab_ssh_host'] = '192.168.254.199'   
gitlab_rails['gitlab_shell_ssh_port'] = 10022

- 生效配置
$ docker exec -it gitlab /bin/bash
root@0f5e1874e9a1:/# gitlab-ctl reconfigure


```

```yaml
root@0f5e1874e9a1:/# vi /opt/gitlab/embedded/service/gitlab-rails/config/gitlab.yml
  
  gitlab:                                                                                   
    ## Web server settings (note: host is the FQDN, do not include http://)                 
    host: 192.168.254.199                              
    port: 8080           # 修改为 10080
    https: false
```

```shell
- 重启gitlab服务
root@0f5e1874e9a1:/# gitlab-ctl restart

```



## 2.windows安装git

[下载安装程序](https://git-scm.com/download/win)

我这里选择： Standalone Installer

安装完成后，在开始菜单里找到“Git”->“Git Bash”，蹦出一个类似命令行窗口的东西，就说明Git安装成功！

安装完成后，还需要最后一步设置，在命令行输入：

```shell
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

然后生成密钥对：

```shell
# cmd命令行
> ssh-keygen

- 然后进入密钥对存放目录：C:\Users\用户名\.ssh
将 id_rsa.pub 存放到 github远程仓库

```

