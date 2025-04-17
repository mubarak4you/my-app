Summary:
Collected baseline resource usage for sysdig-agent and sysdig-clustershield as part of the Sysdig deployment evaluation:

    sysdig-agent

        CPU: 50–70 mCores (Peak: 150 mCores)

        Memory: 373 MiB

    sysdig-clustershield

        CPU: 2.54 Cores

        Memory: 149 MiB

I looked into the best ways to stress test each component based on what they’re designed to do. For sysdig-agent, I’ll be simulating a busy environment by quickly creating and deleting a large number of pods. This will help test how well the agent handles real-world activity where workloads are constantly starting and stopping.

For sysdig-clustershield, I’ll be creating Kubernetes resources that are purposely misconfigured or break best practices. This is to test how well it detects security and compliance issues in the cluster.

The plan is to set up a new test cluster today where I can safely run these tests and observe how both components perform under pressure.