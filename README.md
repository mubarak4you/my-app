clustershield
for i in $(seq 1 200); do touch /etc/shadow-$i /tmp/fuzz-$i; echo "echo test $i" | sh & nc -z 1.1.1.1 80 & dd if=/dev/zero of=/tmp/test-$i bs=1M count=1 & done; sha1sum /dev/zero &


agent
for i in $(seq 1 200); do dd if=/dev/zero of=/tmp/test-$i bs=1M count=1 & sha256sum /dev/zero & yes > /dev/null & done
