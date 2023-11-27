1. 获取当前分支的 commit id
```
git rev-parse --short HEAD  # 短
git rev-parse HEAD  # 完整
```
2. clone 代码到指定文件夹
```
git clone https://github.com/xiaoyekanren/scripts.git [path]
```
3. clone 指定分支
```
git clone -b master https://github.com/xiaoyekanren/scripts.git
```
4. tag
```
git tag -d [tag_name]  # 删除本地tag
git push origin :refs/tags/[tag_name]  # 删除remote的tag
```
5. git add 的反义词
```
git restore --staged xxx.file
git rm --cached *
```
6. 修改commit信息
```
# git commit写错了
git commit --amend
```
7. 删除 git checkout 之后残留的跟当前分支无关的文件夹
```
# 清理无法删除的target文件夹
find ./ -type d -name target | xargs rm -rf
# git clean，-d删除文件夹，-f强制
git clean -df

```
8. 代理
```
# 给当前仓库
git config https.proxy http://127.0.0.1:7890
git config http.proxy http://127.0.0.1:7890
# 全局设置
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
# 取消当前仓库
git config --unset http.proxy
git config --unset https.proxy
# 取消全局
git config --global --unset http.proxy
git config --global --unset https.proxy
```

