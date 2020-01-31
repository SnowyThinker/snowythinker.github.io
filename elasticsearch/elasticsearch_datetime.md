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
~~~java
@Field(type=FieldType.date)
@JsonFormat(pattern="yyyy-MM-dd HH:mm:ss.SSS")
@JsonSerialize(using=LocalDateTimeSerializer.class)
@JsonDeserialize(using=LocalDateTimeDeserializer.class)
private LocalDateTime createTime;
~~~

#### QueryBuilder 映射

~~~java
BoolQueryBuilder queryBuilder = QueryBuilders.boolQuery();
queryBuilder.must(QueryBuilders.rangeQuery("time").gte(beginDate).lte(endDate));
~~~