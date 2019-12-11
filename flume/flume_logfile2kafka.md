# Flume 收集日志发送到Kafka

#### 1、下载 apache flume

#### 2、创建配置文件

~~~shell
touch logfile2kafka.properties
~~~

~~~properties
Flume2KafkaAgent.sources=mysource
Flume2KafkaAgent.channels=mychannel
Flume2KafkaAgent.sinks=mysink

Flume2KafkaAgent.sources.mysource.type=exec
Flume2KafkaAgent.sources.mysource.channels=mychannel
Flume2KafkaAgent.sources.mysource.command=tail -f /data/logs.log

Flume2KafkaAgent.sinks.mysink.channel=mychannel
Flume2KafkaAgent.sinks.mysink.type=org.apache.flume.sink.kafka.KafkaSink
Flume2KafkaAgent.sinks.mysink.kafka.bootstrap.servers=master:9092,slave1:9092
Flume2KafkaAgent.sinks.mysink.kafka.topic=log_topic
Flume2KafkaAgent.sinks.mysink.kafka.batchSize=20
Flume2KafkaAgent.sinks.mysink.kafka.producer.requiredAcks=1

Flume2KafkaAgent.channels.mychannel.type=memory
Flume2KafkaAgent.channels.mychannel.capacity=30000
Flume2KafkaAgent.channels.mychannel.transactionCapacity=100
~~~

#### 3、启动收集进程

~~~shell
# 此处 -n 后的Flume2KafkaAgent 必需和properties配置文件的前缀一样才能生效
nohup ../bin/flume-ng agent -c conf -f conf/flume2kafka.properties -n Flume2KafkaAgent -Dflume.root.logger=INFO,console > /dev/null &
~~~
