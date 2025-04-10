for i in {1..500}; do
  curl "http://<prometheus-server>:9090/api/v1/query?query=rate(container_cpu_usage_seconds_total[5m])" &
done
