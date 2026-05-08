# iotdb-sync
## sync

1. 接收端 启动pipeserver
``` shell
start pipeserver
```
2. 发送端 创建pipesink
``` shell
CREATE PIPESINK node AS IoTDB (ip='172.20.31.23',port=6670)
```
3. 发送端 创建pipe
``` shell
create pipe send_to_23 to node23 FROM (select ** from root) with SyncDelOp=true
```
4. 发送端 启动pipe
``` shell
start pipe send_to_23
```