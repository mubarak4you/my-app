Prometheus-server
for i in {1..50}; do
  curl "http://<prometheus-server>:9090/api/v1/query?query=up" &
done

Load the /api/v1/query endpoint with many complex queries.
