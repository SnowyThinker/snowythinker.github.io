## Elasticsearch sum count group by 按时间区间统计金额与数量

~~~sql
select count(id) totalCount, sum(orderAmount) totalAmount, to_chat(createTime, 'yyyyMMdd') day
group by to_chat(createTime, 'yyyyMMdd')
~~~

#### Java 查询
~~~java
DateHistogramAggregationBuilder dateHistAgg = AggregationBuilders.dateHistogram("day").field("createTime").dateHistogramInterval(DateHistogramInterval.DAY);

ValueCountAggregationBuilder countAgg = AggregationBuilders.count("totalCount").field("id");
SumAggregationBuilder sumAgg = AggregationBuilders.sum("totalAmount").field("orderAmount");

# 添加 count 子聚合
dateHistAgg.subAggregation(countAgg);
# 添加 sum 子聚合
dateHistAgg.subAggregation(sumAgg);

SeqrchQuery searchQuery = new NativeSearchQueryBuilder().withQuery(queryBuilder).addAggregation(dateHistAgg).build();

List<InternalDateHistogram> result = elasticsearchTemplate.query(searchQuery, new ResultExtractor() {

    public Object extract(SearchResponse response) {
        return response.getAggregations().asList();
    }
});

if(null == result || result.isEmpty()) {
    return new ArrayList<>();
}

InternalDateHistogram dateHistogram = result.get(0);
List<Bucket> bucketList = dateHistogram.getBuckets();

bucketList.forEach(action -> {
    Aggregations aggregation = action.getAggregations();

    org.joda.time.DateTime date = (org.joda.time.DateTime)action.getKey();

    InternalSum totalAmount = aggregation.get("totalAmount");
    InternalValueCount totalCount = aggregation.get("totalCount");
});
~~~