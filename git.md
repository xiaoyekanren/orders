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
