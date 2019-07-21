# Spark 官方入门示例

## NetworkWordCount.scala (https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/NetworkWordCount.scala)

## submit task 到 spark

### 用nc启动9000端口做控制台输入功能

~~~sh
nc -lk 9000
~~~

### 打开新shell 窗口切换到 spark bin 目录执行以下命令

~~~sh
./spark-submit --master local[2] \
--class org.apache.spark.examples.streaming.NetworkWordCount \
--name NetworkWordCount \
/app/spark-2.4.3-bin-2.6.0-cdh5.7.0/examples/jars/spark-examples_2.11-2.4.3.jar whoami.centos7 9000
~~~

## 使用Spark shell 方式提交作业

### 打开spark shell

~~~sh
./spark-shell whoami.centos7 local[2]
~~~

~~~java
import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}

val sparkConf = new SparkConf()
sparkConf.setAppName("NetworkWordCount")
sparkConf.set("spark.driver.allowMultipleContexts", "true")
val ssc = new StreamingContext(sparkConf, Seconds(1))
val lines = ssc.socketTextStream("whoami.centos7", 9000)
val words = lines.flatMap(_.split(" "))
val wordCounts = words.map(x => (x, 1)).reduceByKey(_ + _)
wordCounts.print()
ssc.start()
ssc.awaitTermination()
~~~
