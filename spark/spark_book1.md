# Spark高级数据分析

#### 下载示例文件
~~~sh
wget http://bit.ly/1Aoywaq
~~~

#### 将下载好的文件放入 HDFS
~~~sh
# 创建目录
hadoop fs -mkdir /linkage
# 查看目录
hadoop fs -ls /
# 将文件放入 linkage 目录
hadoop fs -put /tmp/blocks_*.csv /linkage
~~~

#### 用Spark shell 加载放入HDFS的csv文件
~~~
val rawblocks = sc.textFile("hdfs://master:9000/linkage")

# 显示第一条数据
rawblocks.first

# 获取前10行记录
val head = rawblocks.take(10)
~~~


#### 定义过滤函数
~~~sh
def isHeader(line: String) = line.contains("id_1")

head.filter(isHeader).foreach(println)

head.filterNot(isHeader).length

# scala 允许使用下划线(_)表示匿名函数的参数
head.filter(!isHeader(_)).length
~~~

#### 2.6 把代码从客户端发到集群
~~~sh
val noheader = rawblocks.filter(x => !isHeader(x))
noheader.first
~~~

#### 2.7 用元组和case class对数据进行结构化
~~~sh
val line = head(5)
val pieces = line.split(',')

# 类型转换
val id1 = pieces(0).toInt
val id2 = pieces(1).toInt

val matched = pieces(11).toBoolean

# 提取数据，用高阶函数map做转换
val rawscores = pieces.slice(2, 11)

def toDouble(s: String) = {
    if("?".equals(s)) Double.NaN else s.toDouble
} 

val scores = rawscores.map(toDouble)
~~~
