### Elasticsearch JMX 监控

#### 修改配置文件
~~~sh
vim $ELASTICSEARCH_HOME/config/jvm.options
~~~

~~~s
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=9400
-Dcom.sun.management.jmxremote.ssl=false
-Dcom.sun.management.jmxremote.authenticate=false
~~~

#### 重启
~~~sh
kill pid 
vim $ELASTICSEARCH_HOME/bin/elasticsearch -d
~~~

#### 用jvisualvm 连接
~~~
ip:9400
~~~

