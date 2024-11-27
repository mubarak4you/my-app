#!/bin/bash

# Grafana credentials and URL
USERNAME="admin"
PASSWORD="<your-admin-password>"
DASHBOARD_URL="<your-dashboard-full-url>"

# Sequential Requests Function
run_sequential_requests() {
    local request_count=$1
    echo "Starting sequential requests (${request_count} requests)..."
    for i in $(seq 1 $request_count); do
        curl -u "$USERNAME:$PASSWORD" -o /dev/null -s -w "Sequential Request $i: HTTP status: %{http_code}\n" "$DASHBOARD_URL"
    done
    echo "Sequential requests completed."
}

# Concurrent Requests Function
run_concurrent_requests() {
    local request_count=$1
    local concurrency=$2
    echo "Starting concurrent requests (${request_count} requests, ${concurrency} concurrent)..."
    seq 1 $request_count | xargs -n1 -P$concurrency -I{} curl -u "$USERNAME:$PASSWORD" -o /dev/null -s -w "Concurrent Request {}: HTTP status: %{http_code}\n" "$DASHBOARD_URL"
    echo "Concurrent requests completed."
}

# Main script execution
echo "Starting Grafana Stress Test..."

# Configure the number of requests
SEQUENTIAL_REQUEST_COUNT=50   # Number of sequential requests
CONCURRENT_REQUEST_COUNT=100  # Number of total concurrent requests
CONCURRENCY_LEVEL=10          # Number of parallel connections for concurrent requests

# Run stress tests
run_sequential_requests $SEQUENTIAL_REQUEST_COUNT
run_concurrent_requests $CONCURRENT_REQUEST_COUNT $CONCURRENCY_LEVEL

echo "Grafana Stress Test Completed."
