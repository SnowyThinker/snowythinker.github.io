### Webapi Server Promotheus Grafana 监控


#### 部署Premetheus
~~~
# 指定9501端口启动
./prometheus --web.listen-address=":9501" &
~~~


# 部署 node-exporter
https://yunlzheng.gitbook.io/prometheus-book/parti-prometheus-ji-chu/quickstart/prometheus-quick-start/use-node-exporter

~~~
curl -OL https://github.com/prometheus/node_exporter/releases/download/v0.17.0/node_exporter-0.17.0.linux-amd64.tar.gz
tar -xzf node_exporter-0.17.0.linux-amd64.tar.gz

# 指定9500端口启动
./node_exporter --web.listen-address=":9500" &
~~~


#### 从Node Exporter收集监控数据
配置获取 prometheus, node export, kubernetes 的metrics数据抽取
~~~
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['192.168.0.32:9501']
  # 采集node exporter监控数据
  - job_name: 'node'
    static_configs:
      - targets: ['192.168.0.32:9500']
  - job_name: 'kubernetes'
    static_configs:
      - targets: ['192.168.0.32:8080']
  - job_name: 'kubernetes-cAdvisor'
    static_configs:
      - targets: ['192.168.0.32:4194']
~~~
重新启动Prometheus Server