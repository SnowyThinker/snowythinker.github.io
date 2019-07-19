### 安装单机hbase



#### 下载写入环境变量
~~~sh
wget http://archive.cloudera.com/cdh5/cdh/5/hbase-1.2.0-cdh5.7.0.tar.gz

export HBASE_HOME=/app/hbase-1.2.0-cdh5.7.0
export PATH=$HBASE_HOME/bin:$PATH
~~~


~~~sh
vim $HBASE_HOME/conf/hbase-env.sh
export JAVA_HOME=/app/jre1.8.0_131
# 关闭hbase自带zookeeper
export HBASE_MANAGES_ZK=false 
~~~


~~~sh
vim $HBASE_HOME/conf/hbase-site.xml
~~~

~~~xml
<configuration>
        <property>
                <name>hbase.rootdir</name>
                <value>hdfs://whoami.centos7:9000/hbase</value>
        </property>
        <property>
                <name>hbase.cluster.distributed</name>
                <value>true</value>
        </property>
        <property>
                <name>hbase.zookeeper.quorum</name>
                <value>whoami.centos7:2181</value>
        </property>
</configuration>
~~~
(此处地址需要和hadoop core-site.xml 配置一样)


~~~sh
vim $HBASE_HOME/conf/regionservers

# 写入主机名
whoami.centos7


vim $HBASE_HOME/bin/hbase-config.sh

export JAVA_HOME=/app/jre1.8.0_131
~~~


### 启动zookeeper

### 启动hbase
~~~sh
$HBASE_HOME/bin/start-hbase.sh
~~~

### 访问验证
http://192.168.254.200:60010


