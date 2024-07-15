# 在mac上使用非root权限安装node
为了避免出现权限不足，可以将brew 安装的node改为手动安装的

1. 清理node缓存
``` shell
npm config get cache  # 查看位置
rm -rf $(npm config get cache)  # 删除
rm -rf ~/.npmrc  # 删除当前的npm配置文件
```

2. 卸载现在的node，所有的
``` shell
brew search node  # 检查当前安装的node
> 输出结果里面已安装的node后面会有√显示，例如：
> node@20 ✔
brew uninstall node@20  # 卸载node
```

3. 下载node
``` shell
wget https://nodejs.org/dist/v20.12.2/node-v20.12.2-darwin-arm64.tar.gz  # 下载 darwin的arm版本
tar zxf node-v20.12.2-darwin-arm64.tar.gz  # 解压缩
mkdir -p ~/Apps  # 创建文件夹~/Apps
mv node-v20.12.2-darwin-arm64 ~/Apps/  # 移动到这里
```

4. 执行node，确定可用
``` shell
~/Apps/node-v20.12.2-darwin-arm64/bin/node -v
> v20.12.2  # 输出
```

5. 写环境变量
``` shell
# 确认shell是哪种
echo $SHELL
``` 

如果是bash
``` shell
> /bin/bash  # 输出，执行下面
cat << EOF >> ~/.bash_profile
export NODE_HOME=/Users/zzm/Apps/node-v20.12.2-darwin-x64
export PATH=\$NODE_HOME/bin:\$PATH
EOF
```

如果是/bin/zsh
``` shell
> /bin/zsh  # 输出，执行下面
cat << EOF >> ~/.zshrc
export NODE_HOME=/Users/zzm/Apps/node-v20.12.2-darwin-x64
export PATH=\$NODE_HOME/bin:\$PATH
EOF
```

6. 后续升级版本
执行3-5即可