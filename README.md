Clarifying the Purpose of ETOKE-359 – Load Balancer Bandwidth Limit Monitoring

The objective of this ticket is to understand and document how OCI Flexible Load Balancer bandwidth service limits are provisioned, consumed, monitored, and increased across regions.

    How it's calculated: The lb-flexible-bandwidth-sum service limit represents the total bandwidth capacity (in Mbps) allowed across all flexible load balancers within a region.

    How it's consumed: Each flexible LB dynamically scales its bandwidth and contributes to this total. The usage is an aggregate of all deployed flexible LBs in the region.

    Current limits and usage:

        us-ashburn-1:

            Limit: 100,000 Mbps

            Current usage: 44,640 Mbps

            Remaining: 55,360 Mbps

        us-phoenix-1:

            Limit: 5,000 Mbps

            Current usage: 2,280 Mbps

            Remaining: 2,720 Mbps

    How to increase it: Limit increases can be requested through the OCI Console → Limits, Quotas and Usage interface by selecting the region and submitting a request against the lb-flexible-bandwidth-sum entry.

    How to monitor it: This limit is not currently exposed as a metric within the Monitoring service. The only way to track it is via:

        Manual or automated querying of the OCI Limits API

        Periodic checks through the Console

        Custom alerting logic based on polling and thresholds (if needed)

This ticket focuses on documenting this behavior and identifying any gaps in proactive monitoring. Next, we’ll verify with OCI if this usage metric can be exposed through native Monitoring or Events, and define whether alerting automation should be implemented around this limit.