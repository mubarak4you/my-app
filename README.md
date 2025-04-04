After reviewing the setup and running initial tests, it appears that there’s not much benefit in actively stress testing the prometheus-node-exporter. Since it’s a passive metrics collector that simply exposes node-level metrics (CPU, memory, disk, etc.), its resource usage remains very low and consistent under normal conditions. Stressing the pod itself doesn’t reflect actual workloads, as it doesn't perform any significant computation or memory-intensive tasks.

That said, based on the baseline values collected, here are the recommended CPU and memory request/limit values for prometheus-node-exporter:

    CPU

        Request: 2m

        Limit: 4m

    Memory

        Request: 24Mi

        Limit: 50Mi

These values provide a safe buffer over observed usage (~1.23 mCPU and 19.3 MiB memory) while keeping the pod lightweight and scheduler-friendly.