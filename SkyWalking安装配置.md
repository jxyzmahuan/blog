---
title: SkyWalking安装配置
date: 2021-09-16 21:14:13
tags:
    - SkyWalking
    - APM
    - Java
    - 链路追踪
    - 微服务治理
---

# elasticsearch7 安装配置

## 下载 elasticsearch7

```
cd /home
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.12.1-linux-x86_64.tar.gz
tar  -xzvf elasticsearch-7.12.1-linux-x86_64.tar.gz

```

## es不能root启动，增加用户

```
useradd -m elasticsearch
passwd elasticsearch
su elasticsearch
chown -R  elasticsearch:elasticsearch  /home/elasticsearch

```

## 修改es7配置文件

```
cd  /home/elasticsearch/elasticsearch-7.12.1/config
vi elasticsearch.yml
#修改数据保存路径
path.data: /data/elasticsearch
```

## 后台启动ES7

```
cd  /home/elasticsearch/elasticsearch-7.12.1/config
../bin/elasticsearch -d
```

# skywalking OAP Server配置

## 下载skywalking

```
cd /home
wget https://downloads.apache.org/skywalking/8.5.0/apache-skywalking-apm-es7-8.5.0.tar.gz
tar zxvf  apache-skywalking-apm-es7-8.5.0.tar.gz 

```

## 修改OAP Server配置文件

```
cd /home/apache-skywalking-apm-bin-es7/config
vi application.yml
#存储换成 elasticsearch7
storage:
  selector: ${SW_STORAGE:elasticsearch7}
#几天一个index
dayStep: ${SW_STORAGE_DAY_STEP:30} # Represent the number of days in the one minute/hour/day index.
indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:1} # Shard number of new indexes
indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:0} # Replicas number of new indexes
# Super data set has been defined in the codes, such as trace segments.The following 3 config would be improve es performance when storage super size data in es.
superDatasetDayStep: ${SW_SUPERDATASET_STORAGE_DAY_STEP:1} # Represent the number of days in the super size dataset record index, the default value is the same as dayStep when the value is less than 0
superDatasetIndexShardsFactor: ${SW_STORAGE_ES_SUPER_DATASET_INDEX_SHARDS_FACTOR:1} #  This factor provides more shards for the super data set, shards number = indexShardsNumber * superDatasetIndexShardsFactor. Also, this factor effects Zipkin and Jaeger traces.
superDatasetIndexReplicasNumber: ${SW_STORAGE_ES_SUPER_DATASET_INDEX_REPLICAS_NUMBER:0} # Represent the replicas number in the super size dataset record index, the default value is 0.
recordDataTTL: ${SW_CORE_RECORD_DATA_TTL:365} # Unit is day
metricsDataTTL: ${SW_CORE_METRICS_DATA_TTL:365} # Unit is day

```

## 启动OAP Server

```
cd /home/apache-skywalking-apm-bin-es7/config
../bin/startup.sh
```

# skywalking agent配置（在微服务所在服务器）

## 下载skywalking

```
cd /home
wget https://downloads.apache.org/skywalking/8.5.0/apache-skywalking-apm-es7-8.5.0.tar.gz
tar zxvf  apache-skywalking-apm-es7-8.5.0.tar.gz 

```

## 修改agent配置文件

```
cd /home/apache-skywalking-apm-bin-es7/agent/config
vi agent.config
#修改命名空间
# The agent namespace
agent.namespace=${SW_AGENT_NAMESPACE:auyen-dev-namespace}
#修改接收数据接口
# Backend service addresses.
collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES:172.31.58.66:11800}
#日志发送配置
plugin.toolkit.log.grpc.reporter.server_host=${SW_GRPC_LOG_SERVER_HOST:172.31.58.66}
plugin.toolkit.log.grpc.reporter.server_port=${SW_GRPC_LOG_SERVER_PORT:11800}
plugin.toolkit.log.grpc.reporter.max_message_size=${SW_GRPC_LOG_MAX_MESSAGE_SIZE:10485760}
plugin.toolkit.log.grpc.reporter.upstream_timeout=${SW_GRPC_LOG_GRPC_UPSTREAM_TIMEOUT:30}

```

## SpringCloudGateway2.1支持

```
cd  /home/apache-skywalking-apm-bin-es7/agent/optional-plugins
#添加webflux支持
cp apm-spring-webflux-5.x-plugin-8.5.0.jar ../plugins/ 
#添加SpringCloudGateway2.1支持
cp apm-spring-cloud-gateway-2.1.x-plugin-8.5.0.jar ../plugins/ 

```


## 修改jenkins流水线run.sh脚本

```
#!/usr/bin/env bash
# ------------------------------------
# 项目运行脚本
# -----------------------------------
set -e

# 环境变量
# 第一个参数就是工程名称
SERVER_NAME=$1
NOW_DATE=`date +%Y%m%d%H%M`
LOG_HOME=/app/$SERVER_NAME/logs

CUSTOM_JVM_OFFLINE="-server
                    -Xmx2g
                    -Xms2g
                    -XX:+UseG1GC
                    -XX:+PrintGCDetails
                    -XX:MaxGCPauseMillis=200
					-javaagent:/home/apache-skywalking-apm-bin-es7/agent/skywalking-agent.jar=agent.service_name=$SERVER_NAME"

CUSTOM_JVM_ONLINE="-server
                    -Xmx2g
                    -Xms2g
                    -XX:+UseG1GC
                    -XX:+PrintGCDetails
                    -XX:MaxGCPauseMillis=200
					-javaagent:/home/apache-skywalking-apm-bin-es7/agent/skywalking-agent.jar=agent.service_name=$SERVER_NAME"



DEFAULT_JVM_LOG_ARGS="  -Xloggc:$LOG_HOME/$SERVER_NAME.gc.log.$NOW_DATE
                        -XX:ErrorFile=$LOG_HOME/$SERVER_NAME.vmerr.log.$NOW_DATE
                        -XX:HeapDumpPath=$LOG_HOME/$SERVER_NAME.heaperr.log.$NOW_DATE"


WORKDIR=/app/$SERVER_NAME

JAR_NAME=`ls -t $WORKDIR/*.jar | grep $SERVER_NAME | head -1 | awk -F/ '{print $NF}'`

if [ -z $VM_LOG ]; then
    VM_LOG="$DEFAULT_JVM_LOG_ARGS"
fi

if [ "$ENV" = "prod" ]; then
    JVM_ARGS="$CUSTOM_JVM_ONLINE"
elif [ "$ENV" = "test" ]; then
    JVM_ARGS="$CUSTOM_JVM_ONLINE $JACOCO_AGENT"
else
    JVM_ARGS="$CUSTOM_JVM_OFFLINE"
fi

echo "-------------------------部署参数--------------------------"
echo "WORKDIR=$WORKDIR"
echo "SERVER_NAME=$SERVER_NAME"
echo "JAR_NAME=$JAR_NAME"
echo "LOG_HOME=$LOG_HOME"
echo "VM_LOG=$VM_LOG"
echo "JVM_ARGS=$JVM_ARGS"
echo "==================================="

cd $WORKDIR
if [ ! -d "logs" ]; then
	mkdir logs
fi
#kill 原来的工程
killall()
{
	SERVICEPID=$(ps -ef | grep =$SERVER_NAME | grep -v grep | grep -v run.sh | head -1 | awk '{print $2}')
	if [ $SERVICEPID ];then
		echo "kill $SERVICEPID"
		kill -9 $SERVICEPID
	fi
}
killall
sleep 1s
nohup java \
     $VM_LOG $JVM_ARGS \
     -Denvironment=$ENV -Dspring.profiles.active=$ENV \
     -jar $WORKDIR/$JAR_NAME >/dev/null 2>&1 &

echo "#结束部署#"

```

