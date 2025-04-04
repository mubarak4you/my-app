for i in {1..100}; do
  curl -s "http://localhost:9090/api/v1/query_range?query=rate(container_cpu_usage_seconds_total[5m])&start=$(($(date +%s) - 300))&end=$(date +%s)&step=1s" > /dev/null &
done



🔍 What it does:

    Fires 100 concurrent queries

    Each query covers 5 minutes of high-resolution (1s step) data

    Uses a heavy query: rate(container_cpu_usage_seconds_total[5m])

🧠 This loads both:

    CPU (PromQL engine parsing + evaluating)

    Memory (retrieving lots of time series data from TSDB)