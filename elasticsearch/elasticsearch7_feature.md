# Elasticsearch 7特性与语法变更

## 显示开启查询总条数信息

~~~shell
GET twitter/_search
{
    "track_total_hits": true,
     "query": {
        "match" : {
            "message" : "Elasticsearch"
        }
     }
}
~~~
