### Prometheus Baseline (Last Hour)

| Component                  | CPU (mCores) | Memory (MiB) |
|---------------------------|--------------|--------------|
| prometheus-server         | 6.90         | 450          |
| prometheus-alertmanager   | 0.796        | 23.5         |
| prometheus-node-exporter  | 1.23         | 19.3         |

---

### Component Resource Configuration

#### ✅ **prometheus-server**

- **Observation**: Core monitoring service with the highest baseline resource consumption among all components.

**Recommended Resources:**
```yaml
resources:
  limits:
    cpu: "30m"
    memory: "1Gi"
  requests:
    cpu: "15m"
    memory: "800Mi"
```

- **Stress Test**:  
  A performance stress test was conducted by executing 5,000 concurrent queries using a heavy PromQL expression:  
  `rate(container_cpu_usage_seconds_total[5m])`.  
  This query is designed to load both CPU and memory resources significantly by retrieving large volumes of time series data from the TSDB (Time Series Database).  
  The test confirmed the need for elevated memory and CPU limits to handle peak query loads without compromising stability.

---

#### ✅ **prometheus-alertmanager**

- **Stress Test**: 300 alerts sent in 1 minute using a `curl` loop.
- **Post-Test Observations**: Memory and CPU usage increased marginally under burst load.

**Recommended Resources:**
```yaml
resources:
  limits:
    cpu: "3m"
    memory: "50Mi"
  requests:
    cpu: "1m"
    memory: "24Mi"
```

---

#### ✅ **prometheus-node-exporter**

- **Observation**: Lightweight and stable usage; stress testing not applicable due to passive role in metrics collection.

**Recommended Resources:**
```yaml
resources:
  limits:
    cpu: "4m"
    memory: "50Mi"
  requests:
    cpu: "2m"
    memory: "24Mi"
```

---

