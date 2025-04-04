sh -c 'for i in $(seq 1 4); do while :; do :; done & done; sleep 60'


🎯 Goal:

Trigger changes in the metrics exposed by prometheus-node-exporter by generating CPU, memory, or disk load on the same node.



3. prometheus-server
for i in {1..50}; do
  curl "http://<prometheus-server>:9090/api/v1/query?query=up" &
done

Load the /api/v1/query endpoint with many complex queries.