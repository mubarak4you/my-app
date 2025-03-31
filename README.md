---
id: observability
title: Observability
---

OKE clusters come pre-installed with observability tools for logging and monitoring. These can be further configured to meet your specific needs.

## Logging
- OKE uses Fluentd agents to forward logs to on-premise Elastic clusters.
- Platform logs (control plane, worker node, and platform workload pods) are sent to a dedicated Elastic cluster.
- Application users should update Fluentd configuration to send application logs to the Verizon on-premise Elastic clusters.
- If an application team has provisioned their own Elastic cluster outside of the Verizon enterprise process, they should not modify the platform Fluentd configuration.
- In such cases, a separate logging agent should be installed to forward application logs to the team’s own Elastic cluster.

![](../images/arch/oke-logging.png)

## Monitoring
- Each OKE cluster runs a separate instance of Grafana and Prometheus.
- Application teams can leverage these tools to create custom dashboards and alerts based on relevant metrics.
