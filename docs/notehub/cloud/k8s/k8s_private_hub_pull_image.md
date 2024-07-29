---
title: k8s私有仓库拉取镜像添加secrets
date: 2023-02-19 21:29:31
tags: 
  - K8S
---
## 创建一个 Secret 来保存登录信息
```
kubectl create secret docker-registry regcred \
    --docker-server=<your-registry-server> \
    --docker-username=<your-name> \
    --docker-password=<your-pword> \
    --docker-email=<your-email>
```

- `<your-registry-server>` 是你的私有仓库的FQDN.
- `<your-name>` 是你的 Docker 用户名.
- `<your-pword>` 是你的 Docker 密码.
- `<your-email>` 是你的 Docker 邮箱可以不填
- 查看secret `kubectl get secret regsecret -o yaml`
```
查看具体内容需要使用base64 -d secret64
```
## 创建一个deployment使用 Secret
```
spec:
      terminationGracePeriodSeconds: 30
      containers:  
      - name: <appName>
        image: <imageTag>
      imagePullSecrets:
      - name: regcred
```

