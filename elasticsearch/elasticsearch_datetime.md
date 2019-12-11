### Elaticsearch timestamp 与 LocalDateTime 映射

#### Elasticsearch脚本 
 ~~~
# 创建索引
PUT customer_index
{
  "mappings": {
    "customer": {
      "properties": {
        "createTime": {
          "type": "date", 
          "format": "yyyy-MM-dd HH:mm:ss.SSS"
        }
      }
    }
  }
}

#### 查看mapping
GET customer_index/_mapping/customer
~~~

#### java映射
~~~
@Field(type=FieldType.date)
@JsonFormat(pattern="yyyy-MM-dd HH:mm:ss.SSS")
@JsonSerialize(using=LocalDateTimeSerializer.class)
@JsonDeserialize(using=LocalDateTimeDeserializer.class)
private LocalDateTime createTime;
~~~