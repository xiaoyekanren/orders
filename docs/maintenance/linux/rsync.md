# sshpass + rsync 批量同步

sshpass 用于自动输入 SSH 密码，结合 rsync 实现无交互远程同步。

## 安装

``` bash
# Debian/Ubuntu
sudo apt update && sudo apt install sshpass rsync

# CentOS/RHEL
sudo yum install sshpass rsync

# macOS
brew install sshpass rsync
```

## 基本语法

``` bash
sshpass [sshpass选项] rsync [rsync选项] 源路径 远程目标路径
```

## 常用参数

| 工具 | 选项 | 说明 |
|------|------|------|
| sshpass | `-p "密码"` | 明文密码（不推荐） |
| sshpass | `-f 密码文件` | 从文件读取（推荐，权限设 600） |
| rsync | `-a` | 归档模式 |
| rsync | `-v` | 详细输出 |
| rsync | `-z` | 传输压缩 |
| rsync | `-P` | 进度条 + 断点续传 |
| rsync | `--delete` | 删除目标端多余文件 |
| rsync | `-e "ssh -p 端口"` | 指定 SSH 端口 |

## 实用示例

### 同步到远程服务器

``` bash
# 使用密码文件（推荐）
echo "remote_password" > ~/.ssh/remote_pass.txt
chmod 600 ~/.ssh/remote_pass.txt

sshpass -f ~/.ssh/remote_pass.txt rsync -avz /local/dir/ user@remote_ip:/remote/dir/
```

### 从远程拉取

``` bash
sshpass -f ~/.ssh/remote_pass.txt rsync -avz user@remote_ip:/remote/dir/ /local/dir/
```

### 非默认 SSH 端口

``` bash
sshpass -f ~/.ssh/remote_pass.txt rsync -avz -e "ssh -p 2222" /local/dir/ user@remote_ip:/remote/dir/
```

### 首次连接（跳过指纹验证）

``` bash
sshpass -f ~/.ssh/remote_pass.txt rsync -avz -e "ssh -o StrictHostKeyChecking=no -p 22" /local/dir/ user@remote_ip:/remote/dir/
```

### 批量同步多台服务器

``` bash
for i in {184..206}; do
  sshpass -p "password" rsync -avzu -e "ssh -o StrictHostKeyChecking=no -p 22" /etc/openvpn/client root@xx.xxx.xx.$i:/etc/openvpn/
done
```

## 目录尾部斜杠的区别

- 源目录加 `/`：同步目录内内容
- 源目录不加 `/`：同步目录本身

## 安全提醒

- 生产环境禁止使用 `-p "明文密码"`（会暴露在 history 和 ps 中）
- 优先使用 SSH 密钥认证
- 密码文件权限必须设为 600