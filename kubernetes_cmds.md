###### Autoscale : Node
```bash
eksctl get nodegroup --cluster=managed-cluster
eksctl scale nodegroup --cluster=<clusterName> --nodes=<desiredCount> --name=<nodegroupName> [ --nodes-min=<minSize> ] [ --nodes-max=<maxSize> ]
eksctl scale nodegroup --name=managed-ng-1 --cluster=managed-cluster --nodes=4 --nodes-min=3 --nodes-max=5
eksctl scale nodegroup --name=managed-ng-1 --cluster=managed-cluster --nodes=1 --nodes-min=1 --nodes-max=5
eksctl scale nodegroup --name=managed-ng-1 --cluster=managed-cluster --nodes=5 --nodes-min=2 --nodes-max=10
kubectl logs -f deployment/cluster-autoscaler -n kube-system
```

###### Autoscale: Pod
```bash
kubectl autoscale deployment venapi-deployment --cpu-percent=50 --min=1 --max=10 -n venapi
kubectl edit hpa venapi-deployment -n venapi
```
###### Rolling Upgrades:
```bash
kubectl --record deployment.apps/venapi-deployment set image deployment.v1.apps/venapi-deployment nginx=nginx:latest -n venapi
kubectl --record deployment.apps/venapi-deployment set image deployment.v1.apps/venapi-deployment nginx=nginx:1.7 -n venapi
kubectl rollout history deployment.v1.apps/venapi-deployment -n venapi
kubectl rollout undo deployment venapi-deployment --to-revision=7 -n venapi
```

###### Misc:
```bash
kubectl get pod -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName --all-namespaces
kubectl get pod -o=custom-columns=NODE:.spec.nodeName,NAME:.metadata.name --all-namespaces
kubectl get pod --all-namespaces -o wide
kubectl get pod --all-namespaces -o json | jq '.items[] | .spec.nodeName + " " + .status.podIP'
kubectl describe nodes
kubectl get all
kubectl get all -o wide
kubectl get all --all-namespaces
kubectl get events
kubectl config set-context --current --namespace=venapi
kubectl api-resources
```
###### Deployments 
```bash
# List deployments:
kubectl get deploy

# Update a deployment with a manifest file:
kubectl apply -f test.yaml

# Scale a deployment “test” to 3 replicas:
kubectl scale deploy/test --replicas=3

# Watch update status for deployment “test”:
kubectl rollout status deploy/test

# Pause deployment on “test”:
kubectl rollout pause deploy/test

# Resume deployment on “test”:
kubectl rollout resume deploy/test

# View rollout history on “test”:
kubectl rollout history deploy/test

# Undo most recent update on “test”:
kubectl rollout undo deploy/test

# Rollback to specific revision on “test”:
kubectl rollout undo deploy/test --to-revision=1
```

```bash
#!/bin/sh
# based on https://gist.github.com/ipedrazas/9c622404fb41f2343a0db85b3821275d

# delete all evicted pods from all namespaces
kubectl get pods --all-namespaces | grep Evicted | awk '{print $2 " --namespace=" $1}' | xargs kubectl delete pod

# delete all containers in ImagePullBackOff state from all namespaces
kubectl get pods --all-namespaces | grep 'ImagePullBackOff' | awk '{print $2 " --namespace=" $1}' | xargs kubectl delete pod

# delete all containers in ImagePullBackOff or ErrImagePull or Evicted state from all namespaces
kubectl get pods --all-namespaces | grep -E 'ImagePullBackOff|ErrImagePull|Evicted' | awk '{print $2 " --namespace=" $1}' | xargs kubectl delete pod
```


###### Load Testing Errors:
```bash
crashloopbackoff in kubernetes - https://sysdig.com/blog/debug-kubernetes-crashloopbackoff/
evicted in kubernetes - https://stackoverflow.com/questions/46419163/what-will-happen-to-evicted-pods-in-kubernetes
Delete evicted pod using below command,
kubectl get pods --all-namespaces -o json | jq '.items[] | select(.status.reason!=null) | select(.status.reason | contains("Evicted")) | "kubectl delete pods \(.metadata.name) -n \(.metadata.namespace)"' | xargs -n 1 bash -com
```

https://aws.amazon.com/premiumsupport/knowledge-center/eks-cluster-autoscaler-setup/
https://aws.amazon.com/premiumsupport/knowledge-center/eks-worker-node-actions/
https://aws.amazon.com/blogs/containers/cost-optimization-for-kubernetes-on-aws/

####### Kubernetes Objects:
* Pods
* Namespaces
* ReplicationController (Manages Pods)
* Deployment (Manages Pods)
* Replicasets
* StatefulSets
* DaemonSets
* Services
* Ingresses
* horizontalpodautoscalers
* ConfigMaps
* Volumes
* Cronjobs
* Jobs
```bash
kubectl api-resources -o wide
```
| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |


```bash
>123
```
|NAME  |                           SHORTNAMES | APIGROUP             |         NAMESPACED  |KIND                          |  VERBS|
|:-----|:-------------------------------------|:---------------------|:--------------------|:----------------------------|:-------|
|bindings                         |    |                                      |  true       |Binding                       | [create]|
