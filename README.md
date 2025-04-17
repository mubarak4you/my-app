kubectl exec -it -n kube-system reflector-5798754d44-vlhf2 -- sh

for i in $(seq 1 10000); do echo "testdata $i" > /tmp/file-$i.log; done