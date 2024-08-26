---
title: k8s的Dashboard安装
date: 2021-07-08 15:30:25
tags: 
  - K8S
---
# k8s的Dashboard安装

```bash
# step1:
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
vim recommended.yaml
# 修改Service为NodePort类型
---
......
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort  # 加上type=NodePort变成NodePort类型的服务
---

# step2:
# 创建
kubectl apply -f recommended.yaml 

# 查看pod的状态
kubectl get pods -n kubernetes-dashboard -l k8s-app=kubernetes-dashboard

# 查看svc分配的端口(note:访问时需要使用https)
kubectl get svc -n kubernetes-dashboard

# step3:
# 创建一个具有全局所有权限的用户来登录Dashboard：(dashboard-admin.yaml)
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: admin
  namespace: kubernetes-dashboard
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin
  namespace: kubernetes-dashboard

---
# 创建
kubectl apply -f admin.yaml

# 查看secret
kubectl get secret -n kubernetes-dashboard|grep admin-token

# 会生成一串很长的base64后的字符串（echo <token值> | base64 -d ）
kubectl get secret <secret名称> -o jsonpath={.data.token} -n kubernetes-dashboard |base64 -d

# 用上面的 base64 解码后的字符串作为 token 登录 Dashboard 即可
```

