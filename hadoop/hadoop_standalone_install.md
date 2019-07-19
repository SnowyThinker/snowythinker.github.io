### 安装单机hadoop


#### SSH 名密钥登录
~~~sh
ssh-keygen -t rsa
cp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys
~~~

#### 下载文件 
wget http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.7.0.tar.gz

#### 编辑配置文件
~~~sh
vim $HADOOP_HOME/etc/hadoop/hadoop-env.sh 
export JAVA_HOME=/app/jre1.8.0_131
~~~
(此步骤不可省略)

~~~sh
vim $HADOOP_HOME/etc/hadoop/core-site.xml
~~~

~~~xml
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>~/Documents/tmp</value>
        <description>A base for other temporary directories.</description>
    </property>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://whoami.local:9000</value>
    </property>
</configuration>
~~~


vim $HADOOP_HOME/etc/hadoop/hdfs-site.xml
~~~xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
~~~



#### 格式化namenode
~~~sh
cd $HADOOP_HOME/bin/
./hdfs namenode -format
~~~

#### 启动hdfs
~~~sh
cd ../sbin
./start-dfs.sh
~~~

#### 访问页面
http://127.0.0.1:50070



Yarn 配置
~~~xml
cd $HADOOP_HOME/etc/hadoop
cp mapred-site.xml.template mapred-site.xml

vim mapred-site.xml

<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
</configuration>

vim yarn-site.xml

<configuration>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
</configuration>
~~~

#### 启动yarn服务
~~~sh
$HADOOP_HOME/sbin/start-yarn.sh 
~~~