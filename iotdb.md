# iotdb

## 配置文件修改
### 关闭合并
```
IOTDB_HOME="/data/ubuntu/apache-iotdb-0.14.0-SNAPSHOT-all-bin"
CONF_PATH=${IOTDB_HOME}/conf
sed -i -e 's/^# enable_seq_space_compaction=.*/enable_seq_space_compaction=false/' ${CONF_PATH}/iotdb-common.properties
sed -i -e 's/^# enable_unseq_space_compaction=.*/enable_seq_space_compaction=false/' ${CONF_PATH}/iotdb-common.properties
sed -i -e 's/^# enable_cross_space_compaction=.*/enable_seq_space_compaction=false/' ${CONF_PATH}/iotdb-common.properties
```
### 加速合并
```
IOTDB_HOME="/data/ubuntu/apache-iotdb-0.14.0-SNAPSHOT-all-bin"
CONF_PATH=${IOTDB_HOME}/conf
# 最大选择文件数量2
sed -i -e 's/^# max_inner_compaction_candidate_file_num=.*/max_inner_compaction_candidate_file_num=2/' ${CONF_PATH}/iotdb-common.properties
# 合并调度间隔1秒
sed -i -e 's/^# compaction_schedule_interval_in_ms=.*/compaction_schedule_interval_in_ms=1000/' ${CONF_PATH}/iotdb-common.properties
# 合并提交间隔1秒
sed -i -e 's/^# compaction_submission_interval_in_ms=.*/compaction_submission_interval_in_ms=1000/' ${CONF_PATH}/iotdb-common.properties
# 不限制合并速度
sed -i -e 's/^# compaction_write_throughput_mb_per_sec=.*/compaction_write_throughput_mb_per_sec=0/' ${CONF_PATH}/iotdb-common.properties
```

## 文件处理
### 清理全部垃圾文件
```
rm -rf LICENSE NOTICE README.md README_ZH.md RELEASE_NOTES.md docs grafana-metrics-example licenses
# clear bat
find ./ -name "*.bat" | xargs rm
```