# iotdb

## 配置文件修改
### 关闭合并
```shell
IOTDB_HOME="/data/ubuntu/apache-iotdb-0.14.0-SNAPSHOT-all-bin"
CONF_PATH=${IOTDB_HOME}/conf
sed -i -e 's/^# enable_seq_space_compaction=.*/enable_seq_space_compaction=false/' ${CONF_PATH}/iotdb-common.properties
sed -i -e 's/^# enable_unseq_space_compaction=.*/enable_unseq_space_compaction=false/' ${CONF_PATH}/iotdb-common.properties
sed -i -e 's/^# enable_cross_space_compaction=.*/enable_cross_space_compaction=false/' ${CONF_PATH}/iotdb-common.properties
```
### 加速合并
```shell
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
### 快速查看已启用的配置项
```shell
cat conf/iotdb-common.properties | grep -v '^$' | grep -v '^#.*$'
```

## 文件处理
### 清理全部垃圾文件
```shell
rm -rf LICENSE NOTICE README.md README_ZH.md RELEASE_NOTES.md docs grafana-metrics-example licenses
# clear bat
find ./ -name "*.bat" | xargs rm
```
### 干死全部jps进程
```shell
jps | awk '{print $1}'|xargs kill -9
```


## 打包client-p
```shell
# 前置条件
pip install setuptools
pip install wheel

# mvn编译打包
# mvn package -pl client-py -am -DskipTests  # old version
mvn package -pl iotdb-client/client-py -am -DskipTests

# 打whl包
cd iotdb-client/client-py/
python setup.py bdist_wheel --universal

# 安装whl包
cd dist
pip3 install apache_iotdb-*.*.*-*.*-none-any.whl
```

## 打包client-cpp
核心命令
```
./mvnw clean package -P with-tools,with-cpp -pl iotdb-client/client-cpp,example/client-cpp-example -am -DskipTests
```
1. windows

2. mac
```
brew install flex
brew install bison
brew install openssl
brew install boost
```

