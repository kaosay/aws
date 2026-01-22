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

## How to expose your web service running in EKS to external traffic.
```
# 第1步：创建IAM OIDC提供者
eksctl utils associate-iam-oidc-provider --region=ap-south-1 --cluster=igs --approve

# Create IAM role for AWS Load Balancer Controller
eksctl create iamserviceaccount \
  --cluster=test \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::aws:policy/ElasticLoadBalancingFullAccess \
  --approve

# Install AWS Load Balancer Controller
helm repo add eks https://aws.github.io/eks-charts
helm repo update
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=test \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

```
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
  namespace: default
spec:
  selector:
    app: web-app
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
  type: ClusterIP
---
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
    # Optional: SSL certificate
    # alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:region:account:certificate/cert-id
spec:
  rules:
    - host: your-domain.com  # Optional: specify domain
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-service
                port:
                  number: 80
```
#### Complete Example with Deployment
```
# Complete example: deployment + service + ingress
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
        - name: web-container
          image: nginx:latest
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web-app
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-service
                port:
                  number: 80

```

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress-ssl
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 443}]'
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:123456789012:certificate/your-cert-id
    alb.ingress.kubernetes.io/ssl-redirect: '443'
    # Health check configuration
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/healthcheck-interval-seconds: '30'
    alb.ingress.kubernetes.io/healthcheck-timeout-seconds: '5'
    alb.ingress.kubernetes.io/healthy-threshold-count: '2'
    alb.ingress.kubernetes.io/unhealthy-threshold-count: '3'
spec:
  rules:
    - host: api.yourdomain.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-service
                port:
                  number: 80

```
## m5a.xlarge node limit 58 pods，change to 110 pods
```
# 检查当前 VPC CNI 配置
kubectl get daemonset aws-node -n kube-system -o jsonpath='{.spec.template.spec.containers[0].env[?(@.name=="ENABLE_PREFIX_DELEGATION")].value}'

# 如果返回空或 false，需要启用前缀分配
kubectl set env daemonset aws-node -n kube-system ENABLE_PREFIX_DELEGATION=true
```
