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
  id: "vpc-a39840c6"  # (optional, must match VPC ID used for each subnet below)
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
