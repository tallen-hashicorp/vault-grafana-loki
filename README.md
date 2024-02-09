# vault-grafana-loki
Quick guide on using Grafana Loki with Vault audit logs

## Copy Vault Audit Logs
To facilitate the collection of Vault audit logs, copy them to a designated folder where Promtail can pick them up later. In production environments, it's recommended to update the `docker run --name promtail` command to specify the exact location of your Vault audit log file, rather than using a folder. Modify the `-v $(pwd)/vault_audit.log:/vault-audit.log` section of the command accordingly.

## Setup Grafana and Loki
Use Docker to set up Grafana and Loki by executing the following commands:

```bash
docker run --name loki -d -v $(pwd):/mnt/config -p 3100:3100 grafana/loki:2.9.4 -config.file=/mnt/config/loki-config.yaml
docker run --name promtail -d -v $(pwd):/mnt/config -v $(pwd)/vault_audit.log:/vault-audit.log -v $(pwd)/test.json:/test.json -p 9080:9080 --link loki grafana/promtail:2.9.4 -config.file=/mnt/config/promtail-config.yaml
docker run -d -p 3000:3000 --name=grafana --link loki grafana/grafana 
```

## Next, Setup Grafana

1. Log in to Grafana via [http://127.0.0.1:3000/](http://127.0.0.1:3000/). The default username and password are `admin/admin`.
2. Click "Add your first data source".
3. Select Loki.
4. Set the connection to `http://loki:3100`.
5. Click "Save & Test" (at the bottom of the page).
6. Go back to the Grafana home by clicking "Home" on the top left of the page.
7. Click "Create your first dashboard".
8. Click "Add Visualization".
9. Select Loki.
10. In the query section, enter `{job="auditlogs"} | json`.
11. Set a limit to 100.
12. Click "Run Query".