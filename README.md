## Installing

### Create namespace for logging
```
kubectl create ns logging
```

### Install released version using Local Helm Directory

Create namespace 

* Install it:
  - Helm 3
  - helm install elasticsearch ./elasticsearch -f elasticsearch/values.yaml -n logging
  - helm install filebeat ./filebeat -f filebeat/values.yaml -n logging
  - helm install kibana ./kibana -f kibana/values.yaml -n logging

Deploy Kibana Ingress
```
kubectl create -f kibana/kibana-ingress.yml
```


### Once Kibana DashBoard is up and running.

1. Open the menu, then go to Stack Management > Kibana > Index Patterns to create a new index pattern. The Index patterns page appears
2. Click the Create index pattern button to begin.
3. In the Define index pattern window, type "filebeat-*" in the Index pattern text box and click the Next step button
4. In the Configure settings window, select @timestamp from the Time Filter field name dropdown menu and click the Create index pattern button
5. A page with the index pattern details appears. Open the menu, then go to Kibana > Discover to view incoming logs
6. The Discover page provides a realtime view of logs as they are ingested by Elasticsearch from the Kubernetes cluster. The histogram provides a view of log volume over time, which by default, spans the last 15 minutes. The sidebar on the left side of the user interface displays various fields parsed from JSON fields sent by Filebeat to Elasticsearch
7. Use the Filters box to search only for logs arriving from Kibana Pods by filtering for kubernetes.container.name : "kibana". Click the Update button to apply the search filter.
