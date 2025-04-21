clustershield
for i in $(seq 1 200); do touch /etc/shadow-$i /tmp/fuzz-$i; echo "echo test $i" | sh & nc -z 1.1.1.1 80 & dd if=/dev/zero of=/tmp/test-$i bs=1M count=1 & done; sha1sum /dev/zero &


agent
for i in $(seq 1 200); do dd if=/dev/zero of=/tmp/test-$i bs=1M count=1 & sha256sum /dev/zero & yes > /dev/null & done





🔧 Stress Test Summary

This update introduces targeted stress tests for both the Sysdig Agent and Sysdig ClusterShield to validate their performance under high system and event load conditions. Each test generates controlled stress patterns that exercise the monitoring and detection logic specific to each component.
🛡️ ClusterShield Stress Test

Command:

for i in $(seq 1 200); do touch /etc/shadow-$i /tmp/fuzz-$i; echo "echo test $i" | sh & nc -z 1.1.1.1 80 & dd if=/dev/zero of=/tmp/test-$i bs=1M count=1 & done; sha1sum /dev/zero &

What it does:

This one-liner simulates a burst of suspicious activity to trigger ClusterShield's runtime security policies. It repeatedly creates fake files in /etc and /tmp, mimicking tampering with system-sensitive paths. It launches a large number of subshells by echoing commands into sh, which mimics potential malicious process spawning. It initiates outbound network scans using nc to simulate command-and-control behavior. The dd command writes raw data into files, generating disk I/O and memory pressure. Finally, sha1sum /dev/zero & continuously computes hashes on a zeroed device stream to produce sustained CPU usage. The combination of these actions stresses ClusterShield’s detection engine, event queueing, and policy evaluation pipelines.
📈 Sysdig Agent Stress Test

Command:

for i in $(seq 1 200); do dd if=/dev/zero of=/tmp/test-$i bs=1M count=1 & sha256sum /dev/zero & yes > /dev/null & done

What it does:

This one-liner targets the Sysdig Agent's core function of host-level metric collection by generating intense system resource utilization. It writes 200+ megabytes of zero-filled files to /tmp using dd, generating disk I/O and allocating memory. The sha256sum /dev/zero part puts pressure on the CPU through continuous hashing operations. Meanwhile, yes > /dev/null spawns a large number of infinite loops that rapidly print characters, creating constant CPU saturation. This test ensures the agent can correctly capture, process, and forward high-frequency telemetry data such as CPU usage, memory consumption, process activity, and disk operations during peak load.
