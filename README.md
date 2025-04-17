

1. sysdig-agent

How to Stress Test Based on Function:
a. Generate High Container/Pod Churn

    Create and delete many pods rapidly:
    
for i in {1..50}; do
  kubectl run testpod-$i --image=busybox --command -- sleep 10 &
done

This simulates workload churn, which increases the event load for the agent.




2. sysdig-clustershield 
a. Rapidly Create Non-Compliant Resources

for i in {1..50}; do
  kubectl create deployment badpod-$i --image=nginx
done

Clustershield will pick up on these misconfigurations and attempt to report findings.