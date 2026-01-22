# How to initial EKS

## m5a.xlarge node limit 58 pods，change to 110 pods
```
# 检查当前 VPC CNI 配置
kubectl get daemonset aws-node -n kube-system -o jsonpath='{.spec.template.spec.containers[0].env[?(@.name=="ENABLE_PREFIX_DELEGATION")].value}'

# 如果返回空或 false，需要启用前缀分配
kubectl set env daemonset aws-node -n kube-system ENABLE_PREFIX_DELEGATION=true
```
