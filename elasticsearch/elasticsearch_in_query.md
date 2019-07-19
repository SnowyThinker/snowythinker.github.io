## Elasticsearch in 查询

#### SQL条件
~~~sql
select * from order
where staus in ('01', '02', '03')
~~~

#### DSL 查询
~~~json
GET /idx_order/order/_search
{
	"query": {
		"bool": {
			"terms": {
				"status.keyword": [
					"01", "02", "03"
				]
			}
		}
	}
}
~~~

#### Java查询
~~~java
QueryBuilder queryBuilder = QueryBuilders.termsQuery("status.keyword", new String[]{"01", "02", "03"});
~~~