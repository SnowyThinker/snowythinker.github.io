
wget https://www.apache.org/dyn/closer.lua/spark/spark-2.4.3/spark-2.4.3.tgz

#### 升级Maven到3.6.1最新版本

#### 解压源码

#### 编辑pom.xml 添加Repository
~~~
<repository>  
  <id>cloudera-repo</id>  
  <name>Cloudera Repository</name>  
  <url>https://repository.cloudera.com/artifactory/cloudera-repos</url>  
  <releases>  
    <enabled>true</enabled>  
  </releases>  
  <snapshots>  
    <enabled>false</enabled>  
  </snapshots>  
</repository>
~~~

#### 编辑pom.xml 添加profile
~~~
	<profile>  
      <id>cdh5.7.0</id>  
      <properties>  
        <hadoop.version>2.6.0-cdh5.7.0</hadoop.version>  
      </properties>  
    </profile>
~~~

#### 编译
~~~
mvn -Pyarn -Phive -Phive-thriftserver -Pcdh5.7.0 -Dhadoop.version=2.6.0-cdh5.7.0 -DskipTests clean package
~~~


#### 打包
~~~
./dev/make-distribution.sh --name 2.6.0-cdh5.7.0  --tgz -Pcdh5.7.0 -Dhadoop.version=2.6.0-cdh5.7.0 -Phive -Phive-thriftserver -Pyarn 
~~~

#### 拷贝出打好的jar包
spark-2.4.3-bin-2.6.0-cdh5.7.0.tgz


https://blog.csdn.net/leen0304/article/details/79037809

