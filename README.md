# vault-grafana-loki
Quick guide on using Grafana Loki with vault audit logs


## Copy vault audit logs
For this i'm going to copy my vault logs to this folder so that they will be picked up later. In production I recomened updating the `docker run --name promtail` to point to your vault audit log file location rather then this folder. The secion of the command to change is `-v $(pwd)/vault_audit.log:/vault-audit.log`


## Setup GRafana and Loki
For this I will use docker to setup Grafana and Loki, follow the below commands to do this:
```bash
docker run --name loki -d -v $(pwd):/mnt/config  -p 3100:3100 grafana/loki:2.9.4 -config.file=/mnt/config/loki-config.yaml
docker run --name promtail -d -v $(pwd):/mnt/config -v $(pwd)/vault_audit.log:/vault-audit.log -v $(pwd)/test.json:/test.json -p 9080:9080 --link loki grafana/promtail:2.9.4 -config.file=/mnt/config/promtail-config.yaml
docker run -d -p 3000:3000 --name=grafana --link loki grafana/grafana 
```

## Next Setup Grafana

* Login to grafana via [http://127.0.0.1:3000/](http://127.0.0.1:3000/), the default username and password is admin/admin
* Click "Add your first data source"
* Select Loki
* Set connection to `http://loki:3100`
* Click Save & Test (at the bottom of the page)
* Go back to grafana home, click home on the top left of the page
* Click Create your first dashboard
* Click Add Visualization
* Select Loki
* In the query select lable `job` and value `auditlogs`
* Set a limit to 100


curl -G -s "http://127.0.0.1:3100/loki/api/v1/query_range" --data-urlencode "query={job=\"vault_audit_logs\"}" --data-urlencode "start=$(date -u +%s --date='-20h')" --data-urlencode "end=$(date -u +%s)" | jq .
