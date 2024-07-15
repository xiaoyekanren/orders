1. npm设置源
```shell
# 给npm设置国内源
# 该步骤执行完之后，会将源的信息写到 /Users/$(whoami)/.npmrc 下
npm config set registry https://registry.npmmirror.com

# 将国内源重置为默认源
npm config delete registry

# 查看当前npm源
npm config get registry
```


2. electron安装不上

```shell
# 切换electron的源为国内即可，以后下载electron 都会走这个地址
# 参考链接https://www.electronjs.org/zh/docs/latest/tutorial/installation
echo "ELECTRON_MIRROR=\"https://npmmirror.com/mirrors/electron/\"" >> ~/.zshrc
```


3. npm设置代理
``` shell
# 设置代理
npm config set proxy http://127.0.0.1:7890
npm config set https-proxy http://127.0.0.1:7890
# 取消代理
npm config delete proxy
npm config delete https-proxy
```
