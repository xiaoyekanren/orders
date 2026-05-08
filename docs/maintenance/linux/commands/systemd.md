# Systemd 服务配置

## .service 配置文件参数

服务文件分为 `[Unit]`、`[Service]`、`[Install]` 三个区块。

### [Unit] 区块

| 参数 | 作用 | 示例 |
|------|------|------|
| Description | 服务描述 | `Description=Disk Usage Monitor Service` |
| Documentation | 文档链接 | `Documentation=man:df(1)` |
| After | 在指定单元之后启动 | `After=network.target mysql.service` |
| Before | 在指定单元之前启动 | `Before=nginx.service` |
| Requires | 强依赖，失败则停止当前服务 | `Requires=mysql.service` |
| Wants | 弱依赖，失败不影响当前服务 | `Wants=logrotate.service` |
| Conflicts | 冲突关系，禁止同时运行 | `Conflicts=apache2.service` |

### [Service] 区块

| 参数 | 作用 | 示例 |
|------|------|------|
| Type | 服务类型 | `simple`/`forking`/`oneshot` |
| ExecStart | 启动命令 | `ExecStart=/usr/bin/myapp` |
| ExecStop | 停止命令 | `ExecStop=/bin/kill $MAINPID` |
| Restart | 重启策略 | `always`/`on-failure`/`no` |
| RestartSec | 重启间隔 | `RestartSec=5` |
| User | 运行用户 | `User=nobody` |
| WorkingDirectory | 工作目录 | `WorkingDirectory=/opt/app` |

### [Install] 区块

| 参数 | 作用 | 示例 |
|------|------|------|
| WantedBy | 安装到哪个 target | `WantedBy=multi-user.target` |
| RequiredBy | 强依赖此服务 | `RequiredBy=network.target` |

## 常用命令

``` shell
# 重载配置
systemctl daemon-reload

# 启用开机自启
systemctl enable myapp

# 禁用开机自启
systemctl disable myapp

# 查看状态
systemctl status myapp

# 查看日志
journalctl -u myapp
```