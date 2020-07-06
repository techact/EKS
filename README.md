# AWS EKS 
AWS EKS is a managed Kubernetes service.
How does EKS work:

![EKS](https://docs.aws.amazon.com/eks/latest/userguide/images/what-is-eks.png)
##### Installation:

###### Prerequest:
  - [Install AWSCLI](#install-awscli)
  - [Install eksctl](#install-eksctl)
  - [Install kubectl](#install-kubectl)
  
###### Overview:
  - [Cluster Creation](#cluster-creation)
  - [Namespace](#namespace)
  - [Service](#Service)
  - [Deployment](#deployment)
  - [ALB-Ingress](#alb-ingress)
  - [Cluster Autoscaling](#cluster-autoscaling)
  - [Horizontal Pod Autoscaling](#horizontal-pod-autoscaling)
  - [Load Test](#load-test)
  - [Monitoring with Prometheus](#monitoring-with-prometheus)
  - [Dashboard - Grafana](#dashboard-grafana)
  
##### CMD's Used:


```bash
eksctl create cluster -f api_cluster_config.yaml
eksctl get cluster --name=managed-k8s-cluster
kubectl get nodes
kubectl cluster-info
kubectl get svc
kubectl get all --all-namespaces
kubectl config view
eksctl get nodegroup --cluster=managed-k8s-cluster
eksctl get nodegroup --cluster=managed-k8s-cluster --name=managed-ng-1
kubectl config get-contexts
kubectl get namespaces
kubectl apply -f venapi-namespace.yaml 
kubectl get namespaces
kubectl apply -f venapi-service.yaml
kubectl get services
kubectl get services -n venapi
kubectl describe service venapi-service -n venapi
kubectl apply -f venapi-deployment.yaml 
kubectl get deployments -n venapi
kubectl describe deployments venapi-deployment -n venapi
kubectl get pods -n venapi
kubectl describe pods  venapi-deployment-9dc68ccb4-l5tmf -n venapi
curl http://172.20.1.237
kubectl apply -f venapi-ingress.yaml 
kubectl get ingress -n venapi
kubectl describe ingress venapi-ingress -n venapi
kubectl apply -f rbac-role.yaml
kubectl apply -f alb-ingress-controller.yaml
kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o alb-ingress[a-zA-Z0-9-]+)
kubectl get ingress -n venapi
kubectl get pods -n venapi
kubectl logs -f venapi-deployment-9dc68ccb4-l5tmf -n venapi
kubectl logs -f deployment/venapi-deployment --all-containers=true -n venapi
```
### Install AWSCLI
``` sh
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```
[AWSCLI Installation reference](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html)
### Install eksctl
```sh
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Install kubectl
```sh
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.16.8/2020-04-16/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

[YAML Markdown](https://gist.github.com/jonschlinkert/5170877)

##### eksctl cluster setup

# Project tree
```bash
managed-k8s-cluster/
├── alb-ingress-controller.yaml
├── cluster_config.yaml
├── rbac-role.yaml
├── venapi-deployment.yaml
├── venapi-ingress.yaml
├── venapi-namespace.yaml
└── venapi-service.yaml
```
### Cluster Creation
##### Cluster Config:
```yaml
# cluster.yaml
# A cluster with an unmanaged nodegroup and two managed nodegroups.
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: managed-cluster
  region: us-west-2
vpc:
  id: "vpc-aaa11111"  # (optional, must match VPC ID used for each subnet below)
  cidr: "10.0.0.0/16"       # (optional, must match CIDR used by the given VPC)
  subnets:
    public:
      us-west-2a:
        id: "subnet-123456"
        cidr: "10.0.1.0/24" # (optional, must match CIDR used by the given subnet)
      us-west-2b:
        id: "subnet-56789"
        cidr: "10.0.2.0/24"  # (optional, must match CIDR used by the given subnet)
    # must provide 'private' and/or 'public' subnets by availibility zone as shown
    private:
      us-west-2a:
        id: "subnet-913232"
        cidr: "10.0.11.0/24" # (optional, must match CIDR used by the given subnet)
      us-west-2b:
        id: "subnet-2423565"
        cidr: "10.0.12.0/24"  # (optional, must match CIDR used by the given subnet)
managedNodeGroups:
  - name: managed-ng-1
    minSize: 2
    maxSize: 5
    desiredCapacity: 2
    instanceType: c5.large
    privateNetworking: true
    ssh:
      allow: true
      publicKeyName: ec2_private_key
    iam:
      withAddonPolicies:
        externalDNS: true
        certManager: true
        albIngress: true
```
```console
> eksctl create cluster -f api_cluster_config.yaml
 ```
```console
[ℹ]  eksctl version 0.23.0
[ℹ]  using region us-west-2
[✔]  using existing VPC (vpc-aaa11111) and subnets (private:[subnet-913232 subnet-2423565] public:[subnet-123456 subnet-56789])
[!]  custom VPC/subnets will be used; if resulting cluster doesn't function as expected, make sure to review the configuration of VPC/subnets
[ℹ]  using EC2 key pair "ec2_private_key"
[ℹ]  using Kubernetes version 1.16
[ℹ]  creating EKS cluster "managed-k8s-cluster" in "us-west-2" region with managed nodes
[ℹ]  1 nodegroup (managed-ng-1) was included (based on the include/exclude rules)
[ℹ]  will create a CloudFormation stack for cluster itself and 0 nodegroup stack(s)
[ℹ]  will create a CloudFormation stack for cluster itself and 1 managed nodegroup stack(s)
[ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=us-west-2 --cluster=managed-k8s-cluster'
[ℹ]  CloudWatch logging will not be enabled for cluster "managed-k8s-cluster" in "us-west-2"
[ℹ]  you can enable it with 'eksctl utils update-cluster-logging --region=us-west-2 --cluster=managed-k8s-cluster'
[ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "managed-k8s-cluster" in "us-west-2"
[ℹ]  2 sequential tasks: { create cluster control plane "managed-k8s-cluster", 2 sequential sub-tasks: { no tasks, create managed nodegroup "managed-ng-1" } }
[ℹ]  building cluster stack "eksctl-managed-k8s-cluster-cluster"
[ℹ]  deploying stack "eksctl-managed-k8s-cluster-cluster"
[ℹ]  building managed nodegroup stack "eksctl-managed-k8s-cluster-nodegroup-managed-ng-1"
[ℹ]  deploying stack "eksctl-managed-k8s-cluster-nodegroup-managed-ng-1"
[ℹ]  waiting for the control plane availability...
[✔]  saved kubeconfig as "/home/ubuntu/.kube/config"
[ℹ]  no tasks
[✔]  all EKS cluster resources for "managed-k8s-cluster" have been created
[ℹ]  nodegroup "managed-ng-1" has 2 node(s)
[ℹ]  node "ip-10-0-11-58.us-west-2.compute.internal" is ready
[ℹ]  node "ip-10-0-12-202.us-west-2.compute.internal" is ready
[ℹ]  waiting for at least 2 node(s) to become ready in "managed-ng-1"
[ℹ]  nodegroup "managed-ng-1" has 2 node(s)
[ℹ]  node "ip-10-0-11-58.us-west-2.compute.internal" is ready
[ℹ]  node "ip-10-0-12-202.us-west-2.compute.internal" is ready
[ℹ]  kubectl command should work with "/home/ubuntu/.kube/config", try 'kubectl get nodes'
[✔]  EKS cluster "managed-k8s-cluster" in "us-west-2" region is ready
```
```console
> eksctl get cluster --name=managed-k8s-cluster
```

```console
NAME			VERSION	STATUS	CREATED			VPC		SUBNETS								SECURITYGROUPS
managed-k8s-cluster	1.16	ACTIVE	2020-07-05T08:45:51Z	vpc-aaa11111	subnet-913232,subnet-2423565,subnet-123456,subnet-56789	sg-231321323234343434
```

```console
 > kubectl get nodes
 ```
 
 ```console
NAME                                        STATUS   ROLES    AGE     VERSION
ip-10-0-11-58.us-west-2.compute.internal    Ready    <none>   7m40s   v1.16.8-eks-fd1ea7
ip-10-0-12-202.us-west-2.compute.internal   Ready    <none>   7m38s   v1.16.8-eks-fd1ea7
```
```bash
> kubectl cluster-info
Kubernetes master is running at https://fjlskfjklsjff.sdfsd.us-west-2.eks.amazonaws.com
CoreDNS is running at https://fjlskfjklsjff.sdfsd.us-west-2.eks.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

```console
> kubectl get svc
```

```console
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   172.20.0.1   <none>        443/TCP   13m
```

```console
> kubectl get all --all-namespaces
```
```console
NAMESPACE     NAME                           READY   STATUS    RESTARTS   AGE
kube-system   pod/aws-node-khmg2             1/1     Running   0          61m
kube-system   pod/aws-node-x6nm6             1/1     Running   0          61m
kube-system   pod/coredns-5c97f79574-5nl4m   1/1     Running   0          66m
kube-system   pod/coredns-5c97f79574-5wh88   1/1     Running   0          66m
kube-system   pod/kube-proxy-mxhqn           1/1     Running   0          61m
kube-system   pod/kube-proxy-t4klr           1/1     Running   0          61m

NAMESPACE     NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)         AGE
default       service/kubernetes   ClusterIP   172.20.0.1    <none>        443/TCP         66m
kube-system   service/kube-dns     ClusterIP   172.20.0.10   <none>        53/UDP,53/TCP   66m

NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
kube-system   daemonset.apps/aws-node     2         2         2       2            2           <none>          66m
kube-system   daemonset.apps/kube-proxy   2         2         2       2            2           <none>          66m

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2/2     2            2           66m

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-5c97f79574   2         2         2       66m

```
```console
> kubectl config view
```
```console
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://12234452234dfsfgdfgdfdg12233.gr7.us-west-2.eks.amazonaws.com
  name: managed-k8s-cluster.us-west-2.eksctl.io
contexts:
- context:
    cluster: managed-k8s-cluster.us-west-2.eksctl.io
    user: deployment-server@managed-k8s-cluster.us-west-2.eksctl.io
  name: deployment-server@managed-k8s-cluster.us-west-2.eksctl.io
current-context: deployment-server@managed-k8s-cluster.us-west-2.eksctl.io
kind: Config
preferences: {}
users:
- name: deployment-server@managed-k8s-cluster.us-west-2.eksctl.io
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1alpha1
      args:
      - eks
      - get-token
      - --cluster-name
      - managed-k8s-cluster
      - --region
      - us-west-2
      command: aws
      env:
      - name: AWS_STS_REGIONAL_ENDPOINTS
        value: regional
```
```console
> eksctl get nodegroup --cluster=managed-k8s-cluster
```
```console
CLUSTER			NODEGROUP	CREATED			MIN SIZE	MAX SIZE	DESIRED CAPACITY	INSTANCE TYPE	IMAGE ID
managed-k8s-cluster	managed-ng-1	2020-07-05T08:56:54Z	2		5		2			c5.large	
```
```console
> eksctl get nodegroup --cluster=managed-k8s-cluster --name=managed-ng-1
```
```console
CLUSTER			NODEGROUP	CREATED			MIN SIZE	MAX SIZE	DESIRED CAPACITY	INSTANCE TYPE	IMAGE ID
managed-k8s-cluster	managed-ng-1	2020-07-05T08:56:54Z	2		5		2			c5.large	
```
```console
kubectl config get-contexts
```
```console
CURRENT   NAME                                                        CLUSTER                                   AUTHINFO                                                    NAMESPACE
*         deployment-server@managed-k8s-cluster.us-west-2.eksctl.io   managed-k8s-cluster.us-west-2.eksctl.io   deployment-server@managed-k8s-cluster.us-west-2.eksctl.io   
```
### Namespace
```console
> kubectl get namespaces
```
```console
NAME              STATUS   AGE
default           Active   79m
kube-node-lease   Active   79m
kube-public       Active   79m
kube-system       Active   79m
```
```console
> cd managed-k8s-cluster/
> cat venapi-namespace.yaml
```
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: venapi
```
```console
> ~/managed-k8s-cluster# kubectl apply -f venapi-namespace.yaml 
```
```console
namespace/venapi created
```
```console
> ~/managed-k8s-cluster# kubectl get namespaces
NAME              STATUS   AGE
default           Active   83m
kube-node-lease   Active   83m
kube-public       Active   83m
kube-system       Active   83m
venapi            Active   5s
```
### Service
```bash
> ~/managed-k8s-cluster# cat venapi-service.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: venapi-service
  namespace: venapi
spec:
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
  type: NodePort
  selector:
    app: venapi
```
```bash
> ~/managed-k8s-cluster# kubectl apply -f venapi-service.yaml
```
```console
service/venapi-service created
```
```bash
> ~/managed-k8s-cluster# kubectl get services
```
```console
> ~/managed-k8s-cluster# kubectl get services -n venapi
NAME             TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
venapi-service   NodePort   172.20.1.237   <none>        80:31474/TCP   17s
```
```bash
~/managed-k8s-cluster# kubectl describe service venapi-service -n venapi
```
```console
Name:                     venapi-service
Namespace:                venapi
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"venapi-service","namespace":"venapi"},"spec":{"ports":[{"port":80...
Selector:                 app=venapi
Type:                     NodePort
IP:                       172.20.1.237
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31474/TCP
Endpoints:                <none>
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```
### Deployment

```bash
> ~/managed-k8s-cluster# cat venapi-deployment.yaml
```
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: venapi-deployment
  namespace: venapi
spec:
  selector:
    matchLabels:
      app: venapi
  replicas: 3
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: venapi
    spec:
      terminationGracePeriodSeconds: 30
      containers:
      - image: nginx:latest
        imagePullPolicy: Always
        name: nginx
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "100Mi"
            cpu: "300m"
          limits:
            memory: "100Mi"
            cpu: "300m"
```
```bash
> ~/managed-k8s-cluster# kubectl apply -f venapi-deployment.yaml 
```
```console
deployment.apps/venapi-deployment created
```
```bash
~/managed-k8s-cluster# kubectl get deployments -n venapi
```
```console
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
venapi-deployment   3/3     3            3           117s
```
```bash
~/managed-k8s-cluster# kubectl describe deployments venapi-deployment -n venapi
```
```console
Name:                   venapi-deployment
Namespace:              venapi
CreationTimestamp:      Sun, 05 Jul 2020 10:51:25 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 2
                        kubectl.kubernetes.io/last-applied-configuration:
                          {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"venapi-deployment","namespace":"venapi"},"spec":{"replica...
Selector:               app=venapi
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  0 max unavailable, 25% max surge
Pod Template:
  Labels:  app=venapi
  Containers:
   nginx:
    Image:      nginx:latest
    Port:       80/TCP
    Host Port:  0/TCP
    Limits:
      cpu:     300m
      memory:  100Mi
    Requests:
      cpu:        300m
      memory:     100Mi
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   venapi-deployment-9dc68ccb4 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  2m57s  deployment-controller  Scaled up replica set venapi-deployment-66b897f44 to 3
```
```bash
> ~/managed-k8s-cluster# kubectl get pods -n venapi
```
```console
NAME                                READY   STATUS    RESTARTS   AGE
venapi-deployment-9dc68ccb4-l5tmf   1/1     Running   0          22s
venapi-deployment-9dc68ccb4-rmnjr   1/1     Running   0          15s
venapi-deployment-9dc68ccb4-sjpck   1/1     Running   0          19s
```
```bash
~/managed-k8s-cluster# kubectl describe pods  venapi-deployment-9dc68ccb4-l5tmf -n venapi
```
```console
Name:         venapi-deployment-9dc68ccb4-l5tmf
Namespace:    venapi
Priority:     0
Node:         ip-10-0-11-58.us-west-2.compute.internal/10.0.11.58
Start Time:   Sun, 05 Jul 2020 11:03:29 +0000
Labels:       app=venapi
              pod-template-hash=9dc68ccb4
Annotations:  kubernetes.io/psp: eks.privileged
Status:       Running
IP:           10.0.11.91
IPs:
  IP:           10.0.11.91
Controlled By:  ReplicaSet/venapi-deployment-9dc68ccb4
Containers:
  nginx:
    Container ID:   docker://4d17d27c760f915f44b2fdbb33a7d4ba10535722facb2fa670acf7e7ac6f3fd5
    Image:          nginx:latest
    Image ID:       docker-pullable://nginx@sha256:21f32f6c08406306d822a0e6e8b7dc81f53f336570e852e25fbe1e3e3d0d0133
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 05 Jul 2020 11:03:31 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     300m
      memory:  100Mi
    Requests:
      cpu:        300m
      memory:     100Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-vj7qg (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-vj7qg:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-vj7qg
    Optional:    false
QoS Class:       Guaranteed
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                                               Message
  ----    ------     ----  ----                                               -------
  Normal  Scheduled  4m2s  default-scheduler                                  Successfully assigned venapi/venapi-deployment-9dc68ccb4-l5tmf to ip-10-0-11-58.us-west-2.compute.internal
  Normal  Pulling    4m1s  kubelet, ip-10-0-11-58.us-west-2.compute.internal  Pulling image "nginx:latest"
  Normal  Pulled     4m    kubelet, ip-10-0-11-58.us-west-2.compute.internal  Successfully pulled image "nginx:latest"
  Normal  Created    4m    kubelet, ip-10-0-11-58.us-west-2.compute.internal  Created container nginx
  Normal  Started    4m    kubelet, ip-10-0-11-58.us-west-2.compute.internal  Started container nginx
```
#### Check whether deployment is working
 Login to any one of worker node and run the below command
 Note: We have used 'NodePort' service, so our application is not accessible with outside cluster
 
 ```bash
[ec2-user@ip-10-0-11-58 ~]$ curl http://172.20.1.237
```
```console
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

It works with in the cluster itself.

But our application needs to accessible through internet. So we need to setup ingress controller.

### ALB-Ingress

```bash
> ~/managed-k8s-cluster# cat venapi-ingress.yaml
```
```console
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: venapi-ingress
  namespace: venapi
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-west-2:1234567890:certificate/12121221-12121212-1212121212-121-121212
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
    alb.ingress.kubernetes.io/subnets: subnet-123456,subnet-56789
    alb.ingress.kubernetes.io/tags: Environment=prod
spec:
  rules:
    - host: venapi.example.com
      http:
        paths:
          - path: /*
            backend:
              serviceName: venapi-service
              servicePort: 80
```
```bash
> ~/managed-k8s-cluster# kubectl apply -f venapi-ingress.yaml 
```
```console
ingress.extensions/venapi-ingress created
```
```bash
~/managed-k8s-cluster# kubectl get ingress -n venapi
```
```console
NAME             HOSTS                    ADDRESS    PORTS   AGE
venapi-ingress   venapi.example.com                  80      3m31s
```
```bash
~/managed-k8s-cluster# kubectl describe ingress venapi-ingress -n venapi
```
```console
Name:             venapi-ingress
Namespace:        venapi
Address:          
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host                    Path  Backends
  ----                    ----  --------
  venapi.example.com  
                          /*   venapi-service:80 (10.0.11.143:80,10.0.11.91:80,10.0.12.5:80)
Annotations:
  alb.ingress.kubernetes.io/listen-ports:            [{"HTTP": 80}, {"HTTPS":443}]
  alb.ingress.kubernetes.io/scheme:                  internet-facing
  alb.ingress.kubernetes.io/subnets:                 subnet-123456,subnet-56789
  alb.ingress.kubernetes.io/tags:                    Environment=prod
  alb.ingress.kubernetes.io/target-type:             ip
  kubectl.kubernetes.io/last-applied-configuration:  {"apiVersion":"extensions/v1beta1","kind":"Ingress","metadata":{"annotations":{"alb.ingress.kubernetes.io/certificate-arn":"arn:aws:acm:us-west-2:1234567890:certificate/12121221-12121212-1212121212-121-121212","alb.ingress.kubernetes.io/listen-ports":"[{\"HTTP\": 80}, {\"HTTPS\":443}]","alb.ingress.kubernetes.io/scheme":"internet-facing","alb.ingress.kubernetes.io/subnets":"subnet-123456,subnet-56789","alb.ingress.kubernetes.io/tags":"Environment=prod","alb.ingress.kubernetes.io/target-type":"ip","kubernetes.io/ingress.class":"alb"},"name":"venapi-ingress","namespace":"venapi"},"spec":{"rules":[{"host":"venapi.example.com","http":{"paths":[{"backend":{"serviceName":"venapi-service","servicePort":80},"path":"/*"}]}}]}}

  kubernetes.io/ingress.class:                alb
  alb.ingress.kubernetes.io/certificate-arn:  arn:aws:acm:us-west-2:1234567890:certificate/12121221-12121212-1212121212-121-121212
Events:                                       <none>
```

We have configured ALB, but loadbalancer is not generated. We need to follow the below steps to create ALB.

Reference: [Deploy the ALB ingress controller](https://kubernetes-sigs.github.io/aws-alb-ingress-controller/guide/walkthrough/echoserver/)

##### Deploy the ALB ingress controller¶

1. Download the example alb-ingress-manifest locally.
```bash 
wget https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.6/docs/examples/alb-ingress-controller.yaml
wget https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.6/docs/examples/rbac-role.yaml
```
2. Edit the manifest and set the following parameters and environment variables.

* cluster-name`: name of the cluster.

* `AWS_ACCESS_KEY_ID`: access key id that alb controller can use to communicate with AWS. This is only used for convenience of this example. It will keep the credentials in plain text within this manifest. It's recommended a project such as kube2iam is used to resolve access. You will need to uncomment this from the manifest.

    - name: AWS_ACCESS_KEY_ID
      value: KEYVALUE

* `AWS_SECRET_ACCESS_KEY`: secret access key that alb controller can use to communicate with AWS. This is only used for convenience of this example. It will keep the credentials in plain text within this manifest. It's recommended a project such as kube2iam is used to resolve access. You will need to uncomment this from the manifest.

    - name: AWS_SECRET_ACCESS_KEY
      value: SECRETVALUE

3. Deploy the modified alb-ingress-controller.
```bash
> ~/managed-k8s-cluster# kubectl apply -f rbac-role.yaml
```
```console
clusterrole.rbac.authorization.k8s.io/alb-ingress-controller created
clusterrolebinding.rbac.authorization.k8s.io/alb-ingress-controller created
serviceaccount/alb-ingress-controller created
```
```bash
> ~/managed-k8s-cluster# kubectl apply -f alb-ingress-controller.yaml
```
```console
deployment.apps/alb-ingress-controller created
```
    The manifest above will deploy the controller to the kube-system namespace.

4. Verify the deployment was successful and the controller started.

```bash
> ~/managed-k8s-cluster# kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o alb-ingress[a-zA-Z0-9-]+)
```

```console
W0705 11:59:47.078994       1 client_config.go:549] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
-------------------------------------------------------------------------------
AWS ALB Ingress controller
  Release:    v1.1.6
  Build:      git-95ee2ac8
  Repository: https://github.com/kubernetes-sigs/aws-alb-ingress-controller.git
-------------------------------------------------------------------------------

I0705 11:59:50.237506       1 controller.go:121] kubebuilder/controller "level"=0 "msg"="Starting EventSource"  "controller"="alb-ingress-controller" "source"={"Type":{"metadata":{"creationTimestamp":null}}}
I0705 11:59:50.237727       1 controller.go:121] kubebuilder/controller "level"=0 "msg"="Starting EventSource"  "controller"="alb-ingress-controller" "source"={"Type":{"metadata":{"creationTimestamp":null},"spec":{},"status":{"loadBalancer":{}}}}
I0705 11:59:50.237789       1 controller.go:121] kubebuilder/controller "level"=0 "msg"="Starting EventSource"  "controller"="alb-ingress-controller" "source"=
I0705 11:59:50.237943       1 controller.go:121] kubebuilder/controller "level"=0 "msg"="Starting EventSource"  "controller"="alb-ingress-controller" "source"={"Type":{"metadata":{"creationTimestamp":null},"spec":{},"status":{"loadBalancer":{}}}}
I0705 11:59:50.237972       1 controller.go:121] kubebuilder/controller "level"=0 "msg"="Starting EventSource"  "controller"="alb-ingress-controller" "source"=
I0705 11:59:50.238091       1 controller.go:121] kubebuilder/controller "level"=0 "msg"="Starting EventSource"  "controller"="alb-ingress-controller" "source"={"Type":{"metadata":{"creationTimestamp":null}}}
I0705 11:59:50.238335       1 controller.go:121] kubebuilder/controller "level"=0 "msg"="Starting EventSource"  "controller"="alb-ingress-controller" "source"={"Type":{"metadata":{"creationTimestamp":null},"spec":{},"status":{"daemonEndpoints":{"kubeletEndpoint":{"Port":0}},"nodeInfo":{"machineID":"","systemUUID":"","bootID":"","kernelVersion":"","osImage":"","containerRuntimeVersion":"","kubeletVersion":"","kubeProxyVersion":"","operatingSystem":"","architecture":""}}}}
I0705 11:59:50.241113       1 leaderelection.go:205] attempting to acquire leader lease  kube-system/ingress-controller-leader-alb...
I0705 11:59:50.248358       1 leaderelection.go:214] successfully acquired lease kube-system/ingress-controller-leader-alb
I0705 11:59:50.248572       1 recorder.go:53] kubebuilder/manager/events "level"=1 "msg"="Normal"  "message"="alb-ingress-controller-5595759966-pd8k7_05a2a1f9-beb7-11ea-9c10-ae0daee26ce3 became leader" "object"={"kind":"ConfigMap","namespace":"kube-system","name":"ingress-controller-leader-alb","uid":"dasdas-dsdd-dds-dsddsa-dfsertretreytr","apiVersion":"v1","resourceVersion":"18737"} "reason"="LeaderElection"
I0705 11:59:50.348737       1 controller.go:134] kubebuilder/controller "level"=0 "msg"="Starting Controller"  "controller"="alb-ingress-controller"
I0705 11:59:50.448899       1 controller.go:154] kubebuilder/controller "level"=0 "msg"="Starting workers"  "controller"="alb-ingress-controller" "worker count"=1
I0705 11:59:50.627443       1 security_group.go:36] venapi/venapi-ingress: creating securityGroup dssasads-venapi-venapiingr-f14a:managed LoadBalancer securityGroup by ALB Ingress Controller
I0705 11:59:50.747522       1 tags.go:69] venapi/venapi-ingress: modifying tags {  kubernetes.io/ingress-name: "venapi-ingress",  ingress.k8s.aws/cluster: "managed-k8s-cluster",  ingress.k8s.aws/stack: "venapi/venapi-ingress",  ingress.k8s.aws/resource: "ManagedLBSecurityGroup",  Environment: "prod",  kubernetes.io/cluster-name: "managed-k8s-cluster",  kubernetes.io/namespace: "venapi"} on sg-12463234435365465
I0705 11:59:50.859761       1 security_group.go:75] venapi/venapi-ingress: granting inbound permissions to securityGroup sg-12463234435365465: [{    FromPort: 80,    IpProtocol: "tcp",    IpRanges: [{        CidrIp: "0.0.0.0/0",        Description: "Allow ingress on port 80 from 0.0.0.0/0"      }],    ToPort: 80  },{    FromPort: 443,    IpProtocol: "tcp",    IpRanges: [{        CidrIp: "0.0.0.0/0",        Description: "Allow ingress on port 443 from 0.0.0.0/0"      }],    ToPort: 443  }]
I0705 11:59:51.019591       1 loadbalancer.go:203] venapi/venapi-ingress: creating LoadBalancer dssasads-venapi-venapiingr-f14a
I0705 11:59:51.593222       1 loadbalancer.go:220] venapi/venapi-ingress: LoadBalancer dssasads-venapi-venapiingr-f14a created, ARN: arn:aws:elasticloadbalancing:us-west-2:1234567890:loadbalancer/app/dssasads-venapi-venapiingr-f14a/f5db23eec2362133
I0705 11:59:51.593379       1 recorder.go:53] kubebuilder/manager/events "level"=1 "msg"="Normal"  "message"="LoadBalancer dssasads-venapi-venapiingr-f14a created, ARN: arn:aws:elasticloadbalancing:us-west-2:1234567890:loadbalancer/app/dssasads-venapi-venapiingr-f14a/f5db23eec2362133" "object"={"kind":"Ingress","namespace":"venapi","name":"venapi-ingress","uid":"22121324334-423443-34243-234234-23423423","apiVersion":"extensions/v1beta1","resourceVersion":"16038"} "reason"="CREATE"
I0705 11:59:52.745542       1 targetgroup.go:126] venapi/venapi-ingress: creating target group dssasads-sadasdartrytyrtyryuu
I0705 11:59:52.902578       1 targetgroup.go:145] venapi/venapi-ingress: target group dssasads-sadasdartrytyrtyryuu created: arn:aws:elasticloadbalancing:us-west-2:1234567890:targetgroup/dssasads-sadasdartrytyrtyryuu/ffbb8c71a86dc9a3
I0705 11:59:52.918437       1 tags.go:43] venapi/venapi-ingress: modifying tags {  ingress.k8s.aws/cluster: "managed-k8s-cluster",  kubernetes.io/cluster/managed-k8s-cluster: "owned",  Environment: "prod",  kubernetes.io/ingress-name: "venapi-ingress",  ingress.k8s.aws/resource: "venapi/venapi-ingress-venapi-service:80",  ingress.k8s.aws/stack: "venapi/venapi-ingress",  kubernetes.io/namespace: "venapi",  kubernetes.io/service-name: "venapi-service",  kubernetes.io/service-port: "80"} on arn:aws:elasticloadbalancing:us-west-2:1234567890:targetgroup/dssasads-sadasdartrytyrtyryuu/ffbb8c71a86dc9a3
I0705 11:59:53.019053       1 targets.go:93] venapi/venapi-ingress: Adding targets to arn:aws:elasticloadbalancing:us-west-2:1234567890:targetgroup/dssasads-sadasdartrytyrtyryuu/ffbb8c71a86dc9a3: 10.0.11.143:80, 10.0.11.91:80, 10.0.12.5:80
I0705 11:59:53.271512       1 listener.go:110] venapi/venapi-ingress: creating listener 80
E0705 11:59:53.311961       1 controller.go:217] kubebuilder/controller "msg"="Reconciler error" "error"="failed to reconcile listeners due to failed to reconcile rules due to ListenerNotFound: One or more listeners not found\n\tstatus code: 400, request id: 8d92bfe6-6b7c-459e-acce-f33470432420"  "controller"="alb-ingress-controller" "request"={"Namespace":"venapi","Name":"venapi-ingress"}
I0705 11:59:54.651924       1 rules.go:62] venapi/venapi-ingress: creating rule 1 on arn:aws:elasticloadbalancing:us-west-2:1234567890:listener/app/dssasads-venapi-venapiingr-f14a/f5db23eec2362133/c8a2b8d1862e7f13
I0705 11:59:54.685401       1 rules.go:79] venapi/venapi-ingress: rule 1 created with conditions [{    Field: "host-header",    HostHeaderConfig: {      Values: ["venapi.example.com"]    }  },{    Field: "path-pattern",    PathPatternConfig: {      Values: ["/*"]    }  }]
I0705 11:59:54.685451       1 listener.go:110] venapi/venapi-ingress: creating listener 443
I0705 11:59:54.685672       1 recorder.go:53] kubebuilder/manager/events "level"=1 "msg"="Normal"  "message"="rule 1 created with conditions [{    Field: \"host-header\",    HostHeaderConfig: {      Values: [\"venapi.example.com\"]    }  },{    Field: \"path-pattern\",    PathPatternConfig: {      Values: [\"/*\"]    }  }]" "object"={"kind":"Ingress","namespace":"venapi","name":"venapi-ingress","uid":"22121324334-423443-34243-234234-23423423","apiVersion":"extensions/v1beta1","resourceVersion":"16038"} "reason"="CREATE"
I0705 11:59:55.021727       1 rules.go:62] venapi/venapi-ingress: creating rule 1 on arn:aws:elasticloadbalancing:us-west-2:1234567890:listener/app/dssasads-venapi-venapiingr-f14a/f5db23eec2362133/b7e3279371d2cad7
I0705 11:59:55.059273       1 rules.go:79] venapi/venapi-ingress: rule 1 created with conditions [{    Field: "host-header",    HostHeaderConfig: {      Values: ["venapi.example.com"]    }  },{    Field: "path-pattern",    PathPatternConfig: {      Values: ["/*"]    }  }]
I0705 11:59:55.059406       1 recorder.go:53] kubebuilder/manager/events "level"=1 "msg"="Normal"  "message"="rule 1 created with conditions [{    Field: \"host-header\",    HostHeaderConfig: {      Values: [\"venapi.example.com\"]    }  },{    Field: \"path-pattern\",    PathPatternConfig: {      Values: [\"/*\"]    }  }]" "object"={"kind":"Ingress","namespace":"venapi","name":"venapi-ingress","uid":"22121324334-423443-34243-234234-23423423","apiVersion":"extensions/v1beta1","resourceVersion":"16038"} "reason"="CREATE"
I0705 11:59:55.614451       1 instance_attachment_v2.go:192] venapi/venapi-ingress: granting inbound permissions to securityGroup sg-sdsdfsdfq34344: [{    FromPort: 0,    IpProtocol: "tcp",    ToPort: 65535,    UserIdGroupPairs: [{        GroupId: "sg-12463234435365465"      }]  }]
I0705 11:59:55.792098       1 controller.go:236] kubebuilder/controller "level"=1 "msg"="Successfully Reconciled"  "controller"="alb-ingress-controller" "request"={"Namespace":"venapi","Name":"venapi-ingress"}
I0705 11:59:56.630485       1 controller.go:236] kubebuilder/controller "level"=1 "msg"="Successfully Reconciled"  "controller"="alb-ingress-controller" "request"={"Namespace":"venapi","Name":"venapi-ingress"}
```
```bash
> ~/managed-k8s-cluster# kubectl get ingress -n venapi
```
```console
NAME             HOSTS                    ADDRESS                                                                  PORTS   AGE
venapi-ingress   venapi.example.com   2442-venapi-venapiingr-3fgff4234-424gdgtyfgggg.us-west-2.elb.amazonaws.com   80      31m
```

Now we can access our application through https://venapi.example.com

Check application access log
```bash
> ~/managed-k8s-cluster# kubectl get pods -n venapi
```
```console
NAME                                READY   STATUS    RESTARTS   AGE
venapi-deployment-9dc68ccb4-l5tmf   1/1     Running   0          163m
venapi-deployment-9dc68ccb4-rmnjr   1/1     Running   0          163m
venapi-deployment-9dc68ccb4-sjpck   1/1     Running   0          163m
```
Checking logs for single pod
```bash
> ~/managed-k8s-cluster# kubectl logs -f venapi-deployment-9dc68ccb4-l5tmf -n venapi
```
```console
10.0.1.120 - - [05/Jul/2020:13:48:53 +0000] "GET / HTTP/1.1" 200 612 "-" "ELB-HealthChecker/2.0" "-"
10.0.2.125 - - [05/Jul/2020:13:48:59 +0000] "GET / HTTP/1.1" 200 612 "-" "ELB-HealthChecker/2.0" "-"
10.0.2.125 - - [05/Jul/2020:13:49:07 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:75.0) Gecko/20100101 Firefox/75.0" "106.203.56.16"
10.0.1.120 - - [05/Jul/2020:13:49:08 +0000] "GET / HTTP/1.1" 200 612 "-" "ELB-HealthChecker/2.0" "-"
10.0.2.125 - - [05/Jul/2020:13:49:14 +0000] "GET / HTTP/1.1" 200 612 "-" "ELB-HealthChecker/2.0" "-"
10.0.1.120 - - [05/Jul/2020:13:49:23 +0000] "GET / HTTP/1.1" 200 612 "-" "ELB-HealthChecker/2.0" "-"
10.0.2.125 - - [05/Jul/2020:13:49:23 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:75.0) Gecko/20100101 Firefox/75.0" "106.203.56.16"
```
checking logs for across all pods - deployment level
```bash
> ~/managed-k8s-cluster# kubectl logs -f deployment/venapi-deployment --all-containers=true -n venapi
```
```console
10.0.2.125 - - [05/Jul/2020:13:48:59 +0000] "GET / HTTP/1.1" 200 612 "-" "ELB-HealthChecker/2.0" "-"
10.0.2.125 - - [05/Jul/2020:13:49:07 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:75.0) Gecko/20100101 Firefox/75.0" "106.203.56.16"
10.0.1.120 - - [05/Jul/2020:13:49:08 +0000] "GET / HTTP/1.1" 200 612 "-" "ELB-HealthChecker/2.0" "-"
10.0.2.125 - - [05/Jul/2020:13:49:14 +0000] "GET / HTTP/1.1" 200 612 "-" "ELB-HealthChecker/2.0" "-"
10.0.1.120 - - [05/Jul/2020:13:49:23 +0000] "GET / HTTP/1.1" 200 612 "-" "ELB-HealthChecker/2.0" "-"
10.0.2.125 - - [05/Jul/2020:13:49:23 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:75.0) Gecko/20100101 Firefox/75.0" "106.203.56.16"
```
### Cluster Autoscaling
### Horizontal Pod Autoscaling
### Load Test 
### Monitoring with Prometheus
### Dashboard - Grafana
