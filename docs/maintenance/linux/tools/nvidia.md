# nvidia

## 1. 限制功率

1. 临时限制

``` shell
# 给编号0 的显卡限制功率到 400w
sudo nvidia-smi -i 0 -pl 400
# 给编号1 的显卡限制功率到 400w
sudo nvidia-smi -i 1 -pl 400

# -pl 临时限制功率
```

2. 启用持久模式

``` shell
sudo nvidia-smi -i 0 -pm 1
sudo nvidia-smi -i 1 -pm 1

# -pm 1 确保配置持续生效
```

3. 查看功率

``` shell
nvidia-smi -i 0 -q -d POWER
nvidia-smi -i 1 -q -d POWER

# -q -d POWER验证功率设置
```

## 2. 锁定驱动版本

``` shell
apt-mark hold cuda-toolkit-12-9
apt-mark hold nvidia-driver-570-open
```