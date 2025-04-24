for i in $(seq 1 50); do
  kubectl run sysdig-cputest-$i --image=go0v-vzdocker/containers/cicd/httpbin/2.6.0 --restart=Never -- sh -c 'for i in $(seq 1 20); do /bin/true; done; sleep 30'
done
