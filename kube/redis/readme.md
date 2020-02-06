# Setup

`kubectl apply -f redis-cluster.yml`

# Init cluster

`kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 $(kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 ')`

# Show state node

`for x in $(seq 0 5); do echo "redis-cluster-$x"; kubectl exec redis-cluster-$x -- redis-cli role; echo; done`

# Add nodes

`kubectl scale statefulset redis-cluster --replicas=8`

Join new nodes to cluster

`kubectl exec redis-cluster-0 -- redis-cli --cluster add-node $(kubectl get pod redis-cluster-6 -o jsonpath='{.status.podIP}'):6379 $(kubectl get pod redis-cluster-0 -o jsonpath='{.status.podIP}'):6379`

Rebalance the masters

`kubectl exec redis-cluster-0 -- redis-cli --cluster rebalance --cluster-use-empty-masters $(kubectl get pod redis-cluster-0 -o jsonpath='{.status.podIP}'):6379`

# Clean

`kubectl delete statefulset,svc,configmap,pvc -l app=redis-cluster`

