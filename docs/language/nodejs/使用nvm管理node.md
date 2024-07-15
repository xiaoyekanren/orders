# 使用 nvm 管理 node

> 操作系统： m1 pro

1. 清理 node 缓存

```shell
npm config get cache  # 查看位置
sudo rm -rf $(npm config get cache)  # 删除
```

2. 删除使用 brew 安装的 node

```shell
# 显示brew安装的node
brew list | grep node
> node@20
> node@18
# 如上，如果有多个版本，依次删除，没有则跳过该步骤
brew uninstall node@20
brew uninstall node@18

```

3. 删除使用 pkg 安装的 node

```shell
sudo rm -rf /usr/local/bin/corepack
sudo rm -rf /usr/local/bin/node
sudo rm -rf /usr/local/bin/npm
sudo rm -rf /usr/local/bin/npx
sudo rm -rf /usr/local/lib/node_modules
sudo rm -rf /usr/local/include/node
sudo rm -rf /usr/local/share/doc/node
sudo rm -rf /usr/local/share/man/man1/node.1
```

4. 安装 nvm  
   首先，打开科学上网。

```shell
touch ~/.zshrc
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```
   执行完之后，将terminal重启。  
   nvm 就安装好了。  

5. nvm的使用
```shell
# 显示可以安装的全部版本
nvm ls-remote
# 显示各版本的最新版本
nvm ls


# 安装node 20
nvm install v20.13.1
# 安装node 18
nvm install v18.20.2
# 安装node 16
nvm install v16.20.2


# 临时切换到node 20，当前窗口生效
nvm use v20 
> Now using node v20.13.1 (npm v10.5.2)
# 切换到node 18，当前窗口生效
nvm use v18
# 永久切换到v16，后续所有窗口都生效 (vscode重启生效)
nvm alias default v16


# 删除node 18
nvm ls  # 确定一下node 18 的最详细版本，例如是 v18.20.2
nvm uninstall v18.20.2



```

6. 卸载nvm
```shell
# vim ~/.zshrc  # 编辑这个文件，删除如下3行

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion


# 删除nvm的目录
 rm -rf ~/.nvm
```
