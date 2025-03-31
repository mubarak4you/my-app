---
id: observability
title: Observability
---

OKE clusters come pre-installed with observability tools for logging and monitoring. These can be further configured to meet your specific needs.


## Logging
- OKE uses Fluentd agents to forward logs to on-premise Elastic clusters.
- Platform logs (control plane, work node, and platform workload pods) are sent to a dedicated Elastic cluster.
- Application teams are responsible for configuring Fluentd to send application pod logs to their own dedicated Elastic clusters.

![](../images/arch/oke-logging.png)


## Monitoring
- Each OKE cluster runs a separate instance of Grafana and Prometheus.
- Application teams can leverage these tools to create custom dashboards and alerts based on relevant metrics.
