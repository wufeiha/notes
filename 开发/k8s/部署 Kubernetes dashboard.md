**作用：**k8s web 可视化面板

**参考：**https://github.com/AliyunContainerService/k8s-for-docker-desktop

### 启动kubernetes

### 验证kubernetes集群状态

```
kubectl cluster-info
kubectl get nodes
```

### 下载配置文件

```
https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.4/aio/deploy/recommended.yaml
```

### 执行配置文件

```
kubectl create -f kubernetes-dashboard.yaml
```

### 检查 kubernetes-dashboard 应用状态

```
kubectl get pod -n kubernetes-dashboard
```

### 开启 API Server 访问代理

```
kubectl proxy
```

### 通过如下 URL 访问 Kubernetes dashboard

  http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

### 配置控制台访问令牌

1. 对于Mac环境

```
TOKEN=$(kubectl -n kube-system describe secret default| awk '$1=="token:"{print $2}')
kubectl config set-credentials docker-for-desktop --token="${TOKEN}"
echo $TOKEN
```

2. 对于Windows环境

```
$TOKEN=((kubectl -n kube-system describe secret default | Select-String "token:") -split " +")[1]
kubectl config set-credentials docker-for-desktop --token="${TOKEN}"
echo $TOKEN
```

#### 