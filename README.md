for i in {1..100}; do
  curl -s "http://localhost:9090/api/v1/query_range?query=rate(container_cpu_usage_seconds_total[5m])&start=$(date -d '5m ago' +%s)&end=$(date +%s)&step=1s" > /dev/null &
done
