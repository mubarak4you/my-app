Problem

Sysdig Agent pods were entering a Pending state during cluster creation due to CPU resource contention. The original CPU request (250m) exceeded available capacity on nodes where multiple DaemonSets are scheduled concurrently at startup.

    From cluster metrics, the total CPU requests from other platform DaemonSets at bootstrap time is approximately 3.48 cores.

    Node allocatable CPU is approximately 3.9 cores.

    When including the Sysdig agent's original CPU request (0.25 cores), the combined total requested CPU was 3.73 cores.

    This left only 0.17 cores (170m) free, which was not enough for the Sysdig agent to be scheduled (needed 250m).

Changes Made

    Reduced sysdig-agent container CPU request from 250m → 100m

        Based on actual observed usage (~26m), 100m provides a safe buffer.

        100m is well within the available 170m CPU margin on the node.

    No changes made to the init container, as it does not contribute to pod pending during scheduling.

Impact

    Prevents sysdig-agent from going Pending on newly provisioned clusters.

    Ensures proper scheduling alongside other critical DaemonSets during cluster bootstrap.

    No performance degradation expected based on historical CPU usage patterns.