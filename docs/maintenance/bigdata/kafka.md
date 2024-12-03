# kafka


``` shell
# 列出全部topic
./kafka-topics.sh --list --zookeeper 192.168.35.21:2181,192.168.35.22:2181,192.168.35.23:2181,192.168.35.24:2181,192.168.35.25:2181

# 查看指定topic详细信息
./kafka-topics.sh --describe --zookeeper 192.168.35.21:2181,192.168.35.22:2181,192.168.35.23:2181,192.168.35.24:2181,192.168.35.25:2181 --topic TY_PP_KTP_CTY_Decode

# 创建topic
./kafka-topics.sh --create --zookeeper 192.168.35.21:2181,192.168.35.22:2181,192.168.35.23:2181,192.168.35.24:2181,192.168.35.25:2181 --replication-factor 5 --partitions 50 --topic Test10_PP_KTP_CTY_Source

# 删除topic
./kafka-topics.sh --delete --zookeeper 192.168.35.21:2181,192.168.35.22:2181,192.168.35.23:2181,192.168.35.24:2181,192.168.35.25:2181 --topic Test10_PP_KTP_CTY_Source

# 热配置数据保留时间,保留300天
./kafka-configs.sh --zookeeper 192.168.35.21:2181,192.168.35.22:2181,192.168.35.23:2181,192.168.35.24:2181,192.168.35.25:2181 --entity-type topics --entity-name Test10_PP_KTP_CTY_Source --alter --add-config retention.ms=25920000000
```
