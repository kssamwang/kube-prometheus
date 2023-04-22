# kube-prometheus
## �ٷ�Readme
[Readme](https://github.com/prometheus-operator/kube-prometheus/blob/main/README.md)

## ����Ŀʹ�÷���
```sh
git clone https://github.com/kssamwang/kube-prometheus.git
cd kube-prometheus
```

��װ
```sh
# Create the namespace and CRDs, and then wait for them to be availble before creating the remaining resources
kubectl create -f manifests/setup

# Wait until the "servicemonitors" CRD is created. The message "No resources found" means success in this context.
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done

kubectl create -f manifests/
```

�Ƴ�
```sh
kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
```

## k8s��װ����Prometheus
### 1 ����kube-prometheus����װ
ѡ��汾v0.10.0
```sh
git clone -b v0.10.0 https://github.com/prometheus-operator/kube-prometheus.git
cd kube-prometheus
```

��װ
```sh
# Create the namespace and CRDs, and then wait for them to be availble before creating the remaining resources
kubectl create -f manifests/setup

# Wait until the "servicemonitors" CRD is created. The message "No resources found" means success in this context.
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done

kubectl create -f manifests/
```

�Ƴ�
```sh
kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
```

### 2 �鿴��������Ϣ
�鿴CRD���͡�
```sh
kubectl get crd | grep coreos
```

�鿴�ض�CRD������ʵ����
```sh
kubectl get prometheuses -n monitoring
kubectl get servicemonitors -n monitoring
```

�鿴������service��
```sh
kubectl get svc -n monitoring
```

��ѯmonitoring�����ռ����������״̬��
```sh
kubectl get po -n monitoring
```

��ѯ��Running״̬pod�ķ��������Ի����������ʧ�ܵ�ԭ��
```sh
kubectl describe  po prometheus-adapter-7858d4ddfd-55lnq  -n monitoring
```

### 3 ���������辵�� ImagePullBackOff / ErrImageNeverPull
�ο���https://blog.csdn.net/qq_45439217/article/details/123477846
#### 3.1 prometheus-adapter
��ȡ��������ľ��񵽱��أ����ϱ�ǩ
```sh
docker pull lbbi/prometheus-adapter:v0.9.1
docker tag docker.io/lbbi/prometheus-adapter:v0.9.1 k8s.gcr.io/prometheus-adapter:v0.9.1
```

```sh
vi manifests/prometheusAdapter-deployment.yaml
```

�ļ����ݣ�
```
***              �޸�ǰ               ***
 
        image: k8s.gcr.io/prometheus-adapter/prometheus-adapter:v0.9.1
        name: prometheus-adapter
 
***              �޸ĺ�               ***
 
        # image: k8s.gcr.io/prometheus-adapter/prometheus-adapter:v0.9.1
        image: k8s.gcr.io/prometheus-adapter:v0.9.1   #  ���������ǩ��д����ȡ���ؾ���ʧ�ܡ�
        imagePullPolicy: Never                        #  imagePullPolicy: Always ����������ȡ����, ��k8sĬ�ϵ���ȡ��ʽ��
                                                      #  imagePullPolicy: Never �Ӳ�Զ����ȡ����ֻ��ȡ���ؾ���
                                                      #  imagePullPolicy: IfNotPresent ������ȡ���ؾ���
        name: prometheus-adapter 
```

����ִ�и����pod��yaml�ļ���
```sh
kubectl replace -f manifests/prometheusAdapter-deployment.yaml
```

#### 3.2 kube-state-metrics
��ȡ��������ľ��񵽱��أ����ϱ�ǩ
```sh
docker pull bitnami/kube-state-metrics:2.3.0
```

```sh
vi manifests/kubeStateMetrics-deployment.yaml
```

�ļ����ݣ�
```
***              �޸�ǰ               ***
 
        image: k8s.gcr.io/kube-state-metrics/kube-state-metrics:2.3.0
        name: kube-state-metrics
 
***              �޸ĺ�               ***
 
        # image: k8s.gcr.io/kube-state-metrics/kube-state-metrics:2.3.0
        image: docker.io/bitnami/kube-state-metrics:2.3.0
        imagePullPolicy: IfNotPresent
        name: kube-state-metrics
```

����ִ�и����pod��yaml�ļ���
```sh
kubectl replace -f manifests/kubeStateMetrics-deployment.yaml
```