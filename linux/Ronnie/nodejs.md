https://zhuanlan.zhihu.com/p/27449990

## 1.Node.js的安装及环境配置

**nvm是啥？**

不同应用程序需要不同版本的 `Node.js`，切换和安装新版本 `Node.js` 很烦人，还会有莫名其妙的问题。`nvm`就是来解决 `Node.js` 的安装和版本切换等问题



`i.安装nvm`

```shell
$ cd ~/
$ git clone https://github.com/nvm-sh/nvm.git .nvm
$ cd ~/.nvm
git checkout v0.38.0
$ . ./nvm.sh
$ cd ~/
```

`ii.配置全局环境`

```shell
$ nano .bash_profile （写入下面代码）

************
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
************

$ source ~/.bash_profile

```

`iii.nvm常用命令`

```shell
- nvm ls-remote 列出所有可安装的版本
- nvm install <version> 安装指定的版本，如 nvm install v8.14.0
- nvm uninstall <version> 卸载指定的版本
- nvm ls 列出所有已经安装的版本
- nvm use <version> 切换使用指定的版本
- nvm current 显示当前使用的版本
- nvm alias default <version> 设置默认 node 版本
- nvm deactivate 解除当前版本绑定
- nvm 默认是不能删除被设定为 default 版本的 node，特别是只安装了一个 node 的时候，这个时候我们需要先解除当前版本绑定，然后再使用 nvm uninstall <version> 删除


```

`iv.安装 Node.js`

```shell
# 1、安装node.js

$ nvm install v14.17.5

# 2、查看node.js版本

$ node -v 

# 3、切换node.js版本

$ nvm ls (查看所有已经安装的node.js版本)
$ nvm use <版本号> （选择从上面列出的node.js版本号）
```

```shell
# 解决nvm切换node版本后，重新打开终端失效
nvm alias default v16.19.0
```

