# 批量注册 GitLab Runner

## 背景

在 ESXi 上使用 Ubuntu 24.04.3 作为基础系统，拷贝多份作为独立 gitlab-runner。但所有机器的 `system_id` 一致，无法用一个 token 同时添加。

## 解决方案

### 重新生成 machine-id

``` shell
# 1. 查看当前 machine-id
cat /etc/machine-id

# 2. 删除
rm /etc/machine-id

# 3. 重新生成
systemd-machine-id-setup

# 4. 确认重新生成
cat /etc/machine-id
cat /var/lib/dbus/machine-id  # 软链接，来自 /etc/machine-id

# 5. 重启（该值变化会影响 netplan 等系统服务）
reboot
```

### 重新注册 gitlab-runner

``` shell
# 1. 删除 .runner_system_id
rm /etc/gitlab-runner/.runner_system_id

# 2. 删除配置文件中所有 [[runners]] 内容
#    注意：多次添加会有多个，需全部删除

# 【可选】使用命令取消注册
# 获取 runner name
gitlab-runner list 2>&1 | grep -v "^Runtime platform" | grep -v "^Listing" | awk '{print $1}'
# 根据 name 取消注册
gitlab-runner unregister --name <name>

# 3. 重启 gitlab-runner
systemctl restart gitlab-runner
```