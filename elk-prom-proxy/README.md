# ELK stack with Prometheus and proxy example
Deploy this stack with steps below 

1. Create proxy overlay network

```
docker network create -d overlay proxy \
--subnet=10.10.0.0/16 \
--gateway=10.10.0.1 
```

2. Create monitor overlay network

```
docker network create -d overlay monitor \
--subnet=10.11.1.0/16 \
--gateway=10.11.1.1 
```

3. Add config files

```
docker config create logstash.conf logstash.conf
docker config create alert_manager_config alert_manager_config
```

4. Deploy stacks - elk, mon, mon-exp, proxy

```
docker stack deploy -c elk.yml elk
docker stack deploy -c mon.yml mon
docker stack deploy -c mon-exp.yml mon-exp
docker stack deploy -c proxy.yml proxy
```

Note: Running Elasticsearch will require instances with at least 4GB memory

5. Verify Prometheus available on port 9090 

Example sorts: 

```
sort_desc(100.0 - 100 * (node_filesystem_avail{device !~'tmpfs',device!~'by-uuid'} / node_filesystem_size{device !~'tmpfs',device!~'by-uuid'}))

node_filesystem_free{mountpoint=~"/rootfs/cio.*"}

(node_filesystem_size{mountpoint=~"/rootfs/cio.*"} - node_filesystem_free{mountpoint=~"/rootfs/cio.*"} ) / 1024 / 1024
```

6. Verify Grafana available on port 3000 
- Login as admin/admin

- Add new datasource:  Select Configuration/Data Sources. Add source info e.g. Name: c1, Type: Prometheus, URL: http://192.168.1.41:9090, Access: direct

- Add new Dashboard:  Select Home/Import dashboard. Grafana.com dashboard:  https://grafana.com/grafana/dashboards/405, click Load. Select prometheus data source, e.g. Prometheus:  c1
Under node, check list nodes to display metrics

7. Use logfiller to generate log data in container
```
docker service create --name logfiller alpine sh -c "while true; do date; done"
```

8. Verify data getting to Elasticsearch
curl -XGET 'http://localhost:9200/logfiller-*/_search?pretty'

9. Verify Kibana available on port 5601
program: logfiller