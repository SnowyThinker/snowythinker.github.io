### Elasticsearch Aggregation


GET /index/document/_search
~~~json
{
    "query": {
        "bool": {
            "filter": [
                {
                    "term": {
                        "userName.keyword": {
                            "value": "Jack"
                        }
                    }
                },
                {
                    "range": {
                        "createTime": {
                            "from": "2019-01-01 00:00:00",
                            "to": "2019-01-02 00:00:00"
                        }
                    }
                }
                
            ]
        }
    },
    "aggregations": {
        "totalCount": {
            "value_count": {
                "field": "userName.keyword"
            }
        },
        "totalAmount": {
            "sum": {
                "field": "orderAmount"
            }
        }
    }
}
~~~