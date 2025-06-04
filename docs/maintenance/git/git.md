# Git
## 获取当前分支的 commit id
``` shell
git rev-parse --short HEAD  # 短
git rev-parse HEAD  # 完整
```

## clone 代码到指定文件夹
``` shell
git clone https://github.com/xiaoyekanren/scripts.git [path]
```

## clone 指定分支
``` shell
git clone -b master https://github.com/xiaoyekanren/scripts.git
```

## tag
``` shell
git tag -d [tag_name]  # 删除本地tag
git push origin :refs/tags/[tag_name]  # 删除remote的tag
```

## git add 的反义词
``` shell
git restore --staged xxx.file
git rm --cached *
```

## 修改commit信息
``` shell
# git commit写错了
git commit --amend
```

## 删除 git checkout 之后残留的跟当前分支无关的文件夹
``` shell
# 清理无法删除的target文件夹
find ./ -type d -name target | xargs rm -rf
# git clean，-d删除文件夹，-f强制
git clean -df
```

## 代理
``` shell
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

## 新建仓库之后，本地第一次上传
``` shell
git remote add origin https://github.com/xiaoyuekanren/test.git
git add .
git commit -m "test"
git push -u origin master
```

## 删除远程tag
``` shell
# 拉取远程的tag
git pull 
# 将当前tag输出到一个文件，删除需要保留的tag
git tag > abc.txt  
# 批量删除：shell里面写一个while循环
cat abc.txt|while read line;do
git push origin --delete refs/tags/${line}
```

## Mac/Linux的代码拷贝到Windows发现所有文件被修改
``` shell
git config --global core.autocrlf true
```

## 避免每次pull都要密码
在当前仓库执行，pull一次后即可无需密码   
``` shell
git config credential.helper store
```


## fork分支提交的PR有冲突，解决冲突

``` shell
# 添加一个 upsteram 分支
git remote add upstream https://github.com/apache/iotdb.git

# 切换到当前仓库的 master
git checkout master

# 从 upstream 更新 master
git pull upstream master

# 提交从 upstream 的改动到 remote 上
git push origin master

# 切换到提交 pr 的分支，例如 aaa
git checkout aaa

# merge master
git merge master

# push， 此时会失败
git push origin zzm

# 重新拉取，此时冲突
git pull

# 解决冲突

# 再次提交
git push origin zzm
```

