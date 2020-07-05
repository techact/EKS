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
### Install eksctl
```sh
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Install kubectl
```sh
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.16.8/2020-04-16/bin/linux/amd64/kubectl
curl -o kubectl.sha256 https://amazon-eks.s3.us-west-2.amazonaws.com/1.16.8/2020-04-16/bin/linux/amd64/kubectl.sha256
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
mkdir -p $HOME/bin && mv ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bash_profile
kubectl version --short --client
```
