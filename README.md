# aws
aws

## Download AWS CLI v2 Installer
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip
sudo ./aws/install
```
## How to install eksctl
```
# 示例：安装 v0.175.0（替换为最新）
wget "https://github.com/eksctl-io/eksctl/releases/download/v0.175.0/eksctl_Linux_amd64.tar.gz" 

tar xvf eksctl_Linux_amd64.tar.gz -C /usr/local/bin

eksctl version
```

## How to use eksctl
```
eksctl utils associate-iam-oidc-provider \
  --cluster test \
  --region ap-south-1 \
  --approve

aws configure set default.region ap-south-1

eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster test \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --override-existing-serviceaccounts

# Update or install the EBS CSI driver add-on
aws eks create-addon \
  --cluster-name test \
  --addon-name aws-ebs-csi-driver \
  --service-account-role-arn arn:aws:iam::ACCOUNT-ID:role/AmazonEKS_EBS_CSI_DriverRole \
  --region ap-south-1

eksctl update addon \
  --name aws-ebs-csi-driver \
  --cluster test \
  --region ap-south-1 \
  --force  # 强制更新，即使版本相同

eksctl get iamserviceaccount --cluster test --region ap-south-1 --namespace kube-system
```
