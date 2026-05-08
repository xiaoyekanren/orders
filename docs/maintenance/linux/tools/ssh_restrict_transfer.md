# SSH 禁用 SCP、RSYNC 和 SFTP

限制 SSH 用户只能执行命令，禁止文件传输。

## 方法一：修改用户 Shell

### 使用 rbash（受限 bash）

```bash
# 将用户的 shell 改为 rbash
usermod -s /bin/rbash username

# 或者创建用户时指定
useradd -s /bin/rbash username
```

### 自定义受限 Shell

创建一个只允许执行特定命令的脚本：

```bash
# /usr/local/bin/restricted_shell
#!/bin/bash
# 只允许执行的命令列表
allowed_commands="ls cat grep pwd echo"

# 获取用户输入的命令
cmd="$1"

# 检查是否在允许列表中
for allowed in $allowed_commands; do
    if [ "$cmd" = "$allowed" ]; then
        exec "$@"
    fi
done

echo "Command not allowed"
exit 1
```

```bash
chmod +x /usr/local/bin/restricted_shell
usermod -s /usr/local/bin/restricted_shell username
```

## 方法二：修改 sshd_config

### 禁用 SFTP 子系统

编辑 `/etc/ssh/sshd_config`，注释或删除：

```
#Subsystem sftp /usr/lib/openssh/sftp-server
```

### 使用 ForceCommand 限制用户

在 `/etc/ssh/sshd_config` 中添加：

```
Match User username
    ForceCommand /bin/bash
    PermitTTY yes
```

### 禁用端口转发和 X11

```
Match User username
    AllowTcpForwarding no
    X11Forwarding no
    PermitTunnel no
```

## 方法三： authorized_keys 限制

在用户的 `~/.ssh/authorized_keys` 中，在公钥前添加限制：

```
command="/bin/bash",no-port-forwarding,no-X11-forwarding,no-agent-forwarding ssh-rsa AAAA...
```

## 方法四：通过 Group 匹配限制

```bash
# 创建受限用户组
groupadd sftp-disabled

# 将用户加入组
usermod -aG sftp-disabled username
```

在 `/etc/ssh/sshd_config` 中：

```
Match Group sftp-disabled
    ForceCommand /bin/bash
    AllowTcpForwarding no
    X11Forwarding no
```

## 验证

```bash
# 重启 sshd
systemctl restart sshd

# 测试 SCP（应失败）
scp test.txt username@host:/tmp/

# 测试 SFTP（应失败）
sftp username@host

# 测试 SSH（应正常）
ssh username@host
```

## 注意事项

- 方法互斥，选择一种即可
- ForceCommand 会限制所有命令，谨慎使用
- rsync 依赖 SSH，禁用 SCP 后 rsync 也会受影响