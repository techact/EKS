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
| suresh | 123 |

```bash
>123
```
|NAME                              |SHORTNAMES   |APIGROUP                       |NAMESPACED   |KIND                             |VERBS|
|bindings                          |             |                               |true         |Binding                          |[create]|
|componentstatuses                 |cs           |                               |false        |ComponentStatus                  |[get list]|
|configmaps                        |cm           |                               |true         |ConfigMap                        |[create delete deletecollection get list patch update watch]|
|endpoints                         |ep           |                               |true         |Endpoints                        |[create delete deletecollection get list patch update watch]|
|events                            |ev           |                               |true         |Event                            |[create delete deletecollection get list patch update watch]|
|limitranges                       |limits       |                               |true         |LimitRange                       |[create delete deletecollection get list patch update watch]|
|namespaces                        |ns           |                               |false        |Namespace                        |[create delete get list patch update watch]|
|nodes                             |no           |                               |false        |Node                             |[create delete deletecollection get list patch update watch]|
|persistentvolumeclaims            |pvc          |                               |true         |PersistentVolumeClaim            |[create delete deletecollection get list patch update watch]|
|persistentvolumes                 |pv           |                               |false        |PersistentVolume                 |[create delete deletecollection get list patch update watch]|
|pods                              |po           |                               |true         |Pod                              |[create delete deletecollection get list patch update watch]|
|podtemplates                      |             |                               |true         |PodTemplate                      |[create delete deletecollection get list patch update watch]|
|replicationcontrollers            |rc           |                               |true         |ReplicationController            |[create delete deletecollection get list patch update watch]|
|resourcequotas                    |quota        |                               |true         |ResourceQuota                    |[create delete deletecollection get list patch update watch]|
|secrets                           |             |                               |true         |Secret                           |[create delete deletecollection get list patch update watch]|
|serviceaccounts                   |sa           |                               |true         |ServiceAccount                   |[create delete deletecollection get list patch update watch]|
|services                          |svc          |                               |true         |Service                          |[create delete get list patch update watch]|
|mutatingwebhookconfigurations     |             |admissionregistration.k8s.io   |false        |MutatingWebhookConfiguration     |[create delete deletecollection get list patch update watch]|
|validatingwebhookconfigurations   |             |admissionregistration.k8s.io   |false        |ValidatingWebhookConfiguration   |[create delete deletecollection get list patch update watch]|
|customresourcedefinitions         |crd,crds     |apiextensions.k8s.io           |false        |CustomResourceDefinition         |[create delete deletecollection get list patch update watch]|
|apiservices                       |             |apiregistration.k8s.io         |false        |APIService                       |[create delete deletecollection get list patch update watch]|
|controllerrevisions               |             |apps                           |true         |ControllerRevision               |[create delete deletecollection get list patch update watch]|
|daemonsets                        |ds           |apps                           |true         |DaemonSet                        |[create delete deletecollection get list patch update watch]|
|deployments                       |deploy       |apps                           |true         |Deployment                       |[create delete deletecollection get list patch update watch]|
|replicasets                       |rs           |apps                           |true         |ReplicaSet                       |[create delete deletecollection get list patch update watch]|
|statefulsets                      |sts          |apps                           |true         |StatefulSet                      |[create delete deletecollection get list patch update watch]|
|tokenreviews                      |             |authentication.k8s.io          |false        |TokenReview                      |[create]
|localsubjectaccessreviews         |             |authorization.k8s.io           |true         |LocalSubjectAccessReview         |[create]|
|selfsubjectaccessreviews          |             |authorization.k8s.io           |false        |SelfSubjectAccessReview          |[create]|
|selfsubjectrulesreviews           |             |authorization.k8s.io           |false        |SelfSubjectRulesReview           |[create]|
|subjectaccessreviews              |             |authorization.k8s.io           |false        |SubjectAccessReview              |[create]|
|horizontalpodautoscalers          |hpa          |autoscaling                    |true         |HorizontalPodAutoscaler          |[create delete deletecollection get list patch update watch]|
|cronjobs                          |cj           |batch                          |true         |CronJob                          |[create delete deletecollection get list patch update watch]|
|jobs                              |             |batch                          |true         |Job                              |[create delete deletecollection get list patch update watch]|
|certificatesigningrequests        |csr          |certificates.k8s.io            |false        |CertificateSigningRequest        |[create delete deletecollection get list patch update watch]|
|leases                            |             |coordination.k8s.io            |true         |Lease                            |[create delete deletecollection get list patch update watch]|
|eniconfigs                        |             |crd.k8s.amazonaws.com          |false        |ENIConfig                        |[delete deletecollection get list patch create update watch]|
|events                            |ev           |events.k8s.io                  |true         |Event                            |[create delete deletecollection get list patch update watch]|
|ingresses                         |ing          |extensions                     |true         |Ingress                          |[create delete deletecollection get list patch update watch]|
|nodes                             |             |metrics.k8s.io                 |false        |NodeMetrics                      |[get list]
|pods                              |             |metrics.k8s.io                 |true         |PodMetrics                       |[get list]|
|ingresses                         |ing          |networking.k8s.io              |true         |Ingress                          |[create delete deletecollection get list patch update watch]|
|networkpolicies                   |netpol       |networking.k8s.io              |true         |NetworkPolicy                    |[create delete deletecollection get list patch update watch]|
|runtimeclasses                    |             |node.k8s.io                    |false        |RuntimeClass                     |[create delete deletecollection get list patch update watch]|
|poddisruptionbudgets              |pdb          |policy                         |true         |PodDisruptionBudget              |[create delete deletecollection get list patch update watch]|
|podsecuritypolicies               |psp          |policy                         |false        |PodSecurityPolicy                |[create delete deletecollection get list patch update watch]|
|clusterrolebindings               |             |rbac.authorization.k8s.io      |false        |ClusterRoleBinding               |[create delete deletecollection get list patch update watch]|
|clusterroles                      |             |rbac.authorization.k8s.io      |false        |ClusterRole                      |[create delete deletecollection get list patch update watch]|
|rolebindings                      |             |rbac.authorization.k8s.io      |true         |RoleBinding                      |[create delete deletecollection get list patch update watch]|
|roles                             |             |rbac.authorization.k8s.io      |true         |Role                             |[create delete deletecollection get list patch update watch]|
|priorityclasses                   |pc           |scheduling.k8s.io              |false        |PriorityClass                    |[create delete deletecollection get list patch update watch]|
|csidrivers                        |             |storage.k8s.io                 |false        |CSIDriver                        |[create delete deletecollection get list patch update watch]|
|csinodes                          |             |storage.k8s.io                 |false        |CSINode                          |[create delete deletecollection get list patch update watch]|
|storageclasses                    |sc           |storage.k8s.io                 |false        |StorageClass                     |[create delete deletecollection get list patch update watch]|
|volumeattachments                 |             |storage.k8s.io                 |false        |VolumeAttachment                 |[create delete deletecollection get list patch update watch]|
