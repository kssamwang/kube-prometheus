# kube-prometheus
## 官方Readme
[Readme](https://github.com/prometheus-operator/kube-prometheus/blob/main/README.md)

## 本项目使用方法
```sh
git clone https://github.com/kssamwang/kube-prometheus.git
cd kube-prometheus
```

安装
```sh
# Create the namespace and CRDs, and then wait for them to be availble before creating the remaining resources
kubectl create -f manifests/setup

# Wait until the "servicemonitors" CRD is created. The message "No resources found" means success in this context.
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done

kubectl create -f manifests/
```

移除
```sh
kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
```

## k8s安装配置Prometheus
### 1 下载kube-prometheus包安装
选择版本v0.10.0
```sh
git clone -b v0.10.0 https://github.com/prometheus-operator/kube-prometheus.git
cd kube-prometheus
```

安装
```sh
# Create the namespace and CRDs, and then wait for them to be availble before creating the remaining resources
kubectl create -f manifests/setup

# Wait until the "servicemonitors" CRD is created. The message "No resources found" means success in this context.
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done

kubectl create -f manifests/
```

移除
```sh
kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
```

### 2 查看监控组件信息
查看CRD类型。
```sh
kubectl get crd | grep coreos
```

查看特定CRD类型下实例。
```sh
kubectl get prometheuses -n monitoring
kubectl get servicemonitors -n monitoring
```

查看创建的service。
```sh
kubectl get svc -n monitoring
```

查询monitoring命名空间所有组件的状态。
```sh
kubectl get po -n monitoring
```

查询非Running状态pod的方法，可以获得容器启动失败的原因。
```sh
kubectl describe  po prometheus-adapter-7858d4ddfd-55lnq  -n monitoring
```

### 3 解决组件所需镜像 ImagePullBackOff / ErrImageNeverPull
参考：https://blog.csdn.net/qq_45439217/article/details/123477846
#### 3.1 prometheus-adapter
拉取可以替代的镜像到本地，打上标签
```sh
docker pull lbbi/prometheus-adapter:v0.9.1
docker tag docker.io/lbbi/prometheus-adapter:v0.9.1 k8s.gcr.io/prometheus-adapter:v0.9.1
```

```sh
vi manifests/prometheusAdapter-deployment.yaml
```

文件内容：
```
***              修改前               ***
 
        image: k8s.gcr.io/prometheus-adapter/prometheus-adapter:v0.9.1
        name: prometheus-adapter
 
***              修改后               ***
 
        # image: k8s.gcr.io/prometheus-adapter/prometheus-adapter:v0.9.1
        image: k8s.gcr.io/prometheus-adapter:v0.9.1   #  容器镜像标签，写错拉取本地镜像失败。
        imagePullPolicy: Never                        #  imagePullPolicy: Always 总是网络拉取镜像, 是k8s默认的拉取方式。
                                                      #  imagePullPolicy: Never 从不远程拉取镜像，只读取本地镜像。
                                                      #  imagePullPolicy: IfNotPresent 优先拉取本地镜像。
        name: prometheus-adapter 
```

重新执行该组件pod的yaml文件。
```sh
kubectl replace -f manifests/prometheusAdapter-deployment.yaml
```

#### 3.2 kube-state-metrics
拉取可以替代的镜像到本地，打上标签
```sh
docker pull bitnami/kube-state-metrics:2.3.0
```

```sh
vi manifests/kubeStateMetrics-deployment.yaml
```

文件内容：
```
***              修改前               ***
 
        image: k8s.gcr.io/kube-state-metrics/kube-state-metrics:2.3.0
        name: kube-state-metrics
 
***              修改后               ***
 
        # image: k8s.gcr.io/kube-state-metrics/kube-state-metrics:2.3.0
        image: docker.io/bitnami/kube-state-metrics:2.3.0
        imagePullPolicy: IfNotPresent
        name: kube-state-metrics
```

重新执行该组件pod的yaml文件。
```sh
kubectl replace -f manifests/kubeStateMetrics-deployment.yaml
```