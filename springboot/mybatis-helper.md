###mybatis-helper(https://github.com/snowThinker/mybatis-helper)

[![Maven Central](https://img.shields.io/maven-central/v/io.github.snowthinker/mybatis-helper.svg?label=Maven%20Central)](https://mvnrepository.com/artifact/io.github.snowthinker/mybatis-helper)
[![License](https://img.shields.io/badge/license-Apache%202-4EB1BA.svg)](https://www.apache.org/licenses/LICENSE-2.0.html)

## 主要功能

* **MyBatis枚举支持**：支持枚举写入/读取数据库
* **动态获取mapper方法的SQL语句**：可根据动态参数获取 MyBatis 格式化好后带参数的SQL


####如何使用
pom 引入jar包
~~~
<dependency>
	<groupId>io.github.snowthinker</groupId>
	<artifactId>mybatis-helper</artifactId>
	<version>0.0.1-SNAPSHOT</version>
</dependency>
~~~

编写枚举类实现 MyBatisEnum 类
~~~
public enum StockMarket implements MyBatisEnum<StockMarket, Integer>{

	US,
	HK,
	CN;

	@Override
	public Integer getValue() {
		return this.ordinal();
	}
}
~~~

####字段枚举支持
mybatis.xml 添加typeHandler
~~~
<typeHandlers>
    <typeHandler handler="io.github.snowthinker.mh.EnumTypeHandler" javaType="io.github.snowthinkder.mh.test.UserStatus"/>
</typeHandlers>
~~~

####获取 mapper.xml 文件方法 SQL语句
~~~
String sql = MybatisSqlHelper.getMapperSql("oi.github.snowthinker.mapper.UserMapper", "queryById", 324);
System.out.println(sql);
~~~


<div class='reward'>
	<div style='text-align: center; display: inline-block; '><img width='250' height='250' src='https://snowthinker.github.io/res/wechat_pay.jpeg'/> &nbsp; &nbsp;&nbsp; <img width='250' height='250' src='https://snowthinker.github.io/res/alipay.jpeg'/>
	</div>
<div>
