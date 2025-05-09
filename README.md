Hi Guillermo,

Thank you for your response.

As requested, I tested this in the test cluster by draining the node pool nap-e2-standard-2-kb58tZ1d1. I observed that the NAP created a new node pool, and both the Sysdig agent DaemonSet and the test DaemonSet I deployed were scheduled on the new node.

Based on this test, it seems the Sysdig DaemonSet remains in a pending state due to a delay in applying the NAP settings before the DaemonSet is attempted to be deployed. Is it correct to assume that NAP isn't accounting for the Sysdig deployment timing, resulting in the DaemonSet being stuck in Pending?

I've attached the relevant logs/output from the cluster for reference.