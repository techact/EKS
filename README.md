# AWS EKS 
AWS EKS is a managed Kubernetes service.
How does EKS work:

![EKS](https://docs.aws.amazon.com/eks/latest/userguide/images/what-is-eks.png)
### Overview
##### Installation:
###### Prerequest:
  - [Install AWSCLI](#install-awscli)
  - [Install eksctl](#install-eksctl)
  - [Install kubectl](#install-kubectl)

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
> cat venapi-namespace.yaml
```
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: venapi
```
