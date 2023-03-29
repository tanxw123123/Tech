```shell
$ git push origin --delete dev         # 删除远程分支
$ git branch -d dev                    # 删除当前分支
$ git reset --hard HEAD                # 回滚代码
$ git checkout -b dev                  # 创建并切换分支
```

## 1.windows安装git

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

