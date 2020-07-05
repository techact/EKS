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
 eksctl create cluster -f api_cluster_config.yaml
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
```
