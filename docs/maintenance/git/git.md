# Git

## 常规命令

### 1. 获取当前分支的 commit id

``` shell
git rev-parse --short HEAD  # 短
git rev-parse HEAD  # 完整
```

### 2. clone 代码

``` shell
# 到指定文件夹
git clone https://github.com/xiaoyekanren/scripts.git [path]

# 指定分支
git clone -b master https://github.com/xiaoyekanren/scripts.git

```

### 3. 新建仓库之后，本地第一次上传

``` shell
git remote add origin https://github.com/xiaoyuekanren/test.git
git add .
git commit -m "test"
git push -u origin master
```

### 4. tag

``` shell
git tag -d [tag_name]  # 删除本地tag
git push origin :refs/tags/[tag_name]  # 删除remote的tag
```

### 5. 代理

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

## 稍稍进阶

### 1. git add 的反义词

``` shell
git restore --staged xxx.file
git rm --cached *
```

### 2. commit写错了，修改commit信息

``` shell
git commit --amend
```

### 3. 反转 commit

```shell
# HEAD~1 表示撤销最近1次提交
# HEAD~2 表示撤销最近2次（要慎重）
git reset --soft HEAD~1
```

### 4. 完全抛弃掉所有修改

```shell
# 适用于还未add
git reset --hard
```

### 5. 删除远程tag

``` shell
# 拉取远程的tag
git pull 
# 将当前tag输出到一个文件，删除需要保留的tag
git tag > abc.txt  
# 批量删除：shell里面写一个while循环
cat abc.txt|while read line;do
git push origin --delete refs/tags/${line}
```

### 6. git log 一行输出

``` shell
# 格式：日期,Commit-ID,Author,Title
git log --pretty=format:"%ad,%h,%an,%s" --date=format:"%Y-%m-%d %H:%M:%S"

# 这个也行，但是消息很简略
git log --oneline
```

### 7. 添加新的远程分支 upstream

``` shell
# 添加一个 upsteram 分支
git remote add upstream https://github.com/apache/iotdb.git

# 从 upstream 更新 master
git pull upstream master

# 提交从 upstream 的改动到 remote 上
git push origin master
```

## 常见问题

### 1. Mac/Linux的代码拷贝到Windows发现所有文件被修改

``` shell
git config --global core.autocrlf true
```

### 2. 避免每次pull都要密码

在当前仓库执行，pull一次后即可无需密码

``` shell
git config credential.helper store
```

### 3. 删除 git checkout 之后残留的跟当前分支无关的文件夹

``` shell
# git clean，-d删除文件夹，-f强制
git clean -df
```


