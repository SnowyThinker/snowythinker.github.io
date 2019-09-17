# Redis大Key与大Value分析

## 1、大Key分析

### 使用redis-cli 自带命令 

~~~shell
redis-cli -n <db_number> -a <password> --bigkeys
~~~

## 2、大Value分析，按从大到小排序

### 安装redis分析工具 rdb

https://github.com/sripathikrishnan/redis-rdb-tools#generate-memory-report

~~~shell
pip install rdbtools python-lzf
~~~

### 执行分析命令

~~~shell
# rdb与aof文件都可以
rdb -c memory /path/to/your/dump.rdb > result.csv
~~~

### 将CSV转换成 Excel 按 sizes_in_bytes 倒序排列

~~~shell
此步骤省略
~~~
