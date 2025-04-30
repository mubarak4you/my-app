Sysdig Test Plan
Objective:

Evaluate Sysdig agent scheduling behavior under higher CPU request scenarios and validate Node Auto-Provisioning (NAP) response.
Step 1: Baseline Resource Audit

    Identify all DaemonSets with equal or higher priorityClass than the Sysdig agent.

    Calculate the total CPU request from these DaemonSets to assess baseline CPU pressure on new nodes.

Step 2: Simulate High Resource Demand

    Manually increase Sysdig agent CPU request to 2 cores in Helm values or manifest.

    Observe initial pod scheduling behavior:

        Does it remain pending?

        Does it trigger any immediate scaling?

Step 3: Test Node Auto-Provisioning (NAP) Behavior

    Ensure NAP is enabled on the node pool.

    Apply the high-resource Sysdig configuration.

    Allow cluster to reach steady state and observe:

        Whether NAP provisions a new node to accommodate the pod.

        How long it takes for the new node to be created and Sysdig to be scheduled.

Step 4: Controlled Scale-Up Test

    Configure the environment with:

        Sysdig agent using high resource requests (2 CPU).

        Fewer nodes initially to trigger scaling.

    Validate:

        Sysdig eventually schedules only after NAP provisions additional capacity.

        No preemption occurs (unless intentionally configured).

Expected Outcome

    Confirm that with high resource requests, Sysdig does not cause cluster instability.

    Validate that NAP correctly identifies unschedulable pods and adds capacity as expected.

    Assess the time-to-schedule and behavior consistency across reboots or restarts.