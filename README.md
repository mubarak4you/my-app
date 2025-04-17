✅ 1. Sysdig Agent Memory Stress Test (Realistic I/O Workload)

This simulates an app generating a ton of log files or temp data, which triggers file-related syscalls (open, write, close) that Sysdig monitors.

kubectl run sysdig-memtest --image=go0v-vzdocker/containers/cicd/httpbin/2.6.0 --restart=Never -- \
  sh -c 'for i in $(seq 1 10000); do echo "testdata $i" > /tmp/file-$i.log; done; sleep 60'

💡 What this triggers for Sysdig:

    File creation & modification

    Directory traversal

    Open/write/close syscalls

    Audit trail processing

    Activity tracking

✅ 2. Sysdig Agent CPU Stress Test (Realistic Process Activity)

This simulates a service that's constantly launching subprocesses, which Sysdig picks up via execve, clone, exit syscalls.

kubectl run sysdig-cputest --image=go0v-vzdocker/containers/cicd/httpbin/2.6.0 --restart=Never -- \
  sh -c 'for i in $(seq 1 5000); do /bin/true; done; sleep 30'

💡 What this triggers for Sysdig:

    Thousands of short-lived process events

    High execve syscall rate

    Activity audit logging

    Event buffer processing (real CPU pressure point)