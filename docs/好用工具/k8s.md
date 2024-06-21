---
sort: 13
---

# Kubernetes

## 安装


## 脚本

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: rtsb-sub
spec:
  completions: 2
  parallelism: 1
  template:
    metadata:
      labels:
        app: rtsb-sub
    spec:
      containers:
      - name: throughput-sub
        image: rtsb_lite
        imagePullPolicy: Never
        command: ["/bin/bash", "-c", "cd /aout/ && source release.com && ./throughput-sub 30 -1 $((i + 1))"]
        resources:
          limits:
            cpu: "100m"
            memory: "64Mi"
      restartPolicy: Never
---
apiVersion: batch/v1
kind: Job
metadata:
  name: rtsb-pub
spec:
  completions: 2
  parallelism: 1
  template:
    metadata:
      labels:
        app: rtsb-pub
    spec:
      containers:
      - name: throughput-pub
        image: rtsb_lite
        imagePullPolicy: Never
        command: ["/bin/bash", "-c", "cd /aout/ && source release.com && ./throughput-pub 8192 0 1 31 $((i + 1))"]
        resources:
          limits:
            cpu: "100m"
            memory: "64Mi"
      restartPolicy: Never
```

## 一般使用

- 列出节点：`kubectl get node -o wide`（必须有从节点，主节点不能部署）
- 部署容器：`kubectl apply -f rtsb-lite.yaml`
- 删除容器：`kubectl delete -f rtsb-lite.yaml`

- 列出job：`kubectl get job -o wide`
- 查看job的状态和原因：
	`kubectl describe job <job-name>`
	或者使用标签选择器：`kubectl describe jobs -l app=rtsb-pub`
- 删除job：
	- `kubectl delete job --all`
	- `kubectl delete job <job-name>`
	- `kubectl describe job -l app=rtsb-pub`

- 列出pod：`kubectl get pod -o wide`
- 进入pod：`kubectl exec -it <pod-name> -n <namespace> -- bash`
- 获取Pod的当前容器日志：
	- `kubectl logs <pod1>,<my-pod2>`   -f（持续）
	- `kubectl logs -f -l app=rtsb-sub`
- 删除pods：
	- `kubectl delete pod --all`
	- `kubectl delete pod <pod-name>`
	- `kubectl delete pod -l <label-selector>`


问题：
1. 跨主机容器通信？

在Kubernetes中，每个Pod都有一个唯一的IP地址，而Pod内的所有容器共享这个IP地址。这意味着Pod中的每个容器看到的网络接口和IP地址是相同的。您不需要（也无法）为Pod内的每个容器分配单独的IP地址。

使用flannel插件，配置网络为“host-gw”模式：
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        app: flannel
    spec:
      containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.14.0
        command: [ "/opt/bin/flanneld", "--ip-masq", "--kube-subnet-mgr", "--iface=<INTERFACE>" ]
        securityContext:
          privileged: true
        env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
        volumeMounts:
        - name: run
          mountPath: /run
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      - name: install-cni
        image: quay.io/coreos/flannel:v0.14.0
        command: [ "/opt/bin/install-cni", ]
        volumeMounts:
        - name: cni-dir
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      volumes:
        - name: run
          hostPath:
            path: /run
        - name: cni-dir
          hostPath:
            path: /etc/cni/net.d
        - name: flannel-cfg
          configMap:
            name: kube-flannel-cfg
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-flannel-cfg
  namespace: kube-system
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
  net-conf.json: |
    {
      "Network": "10.244.1.0/24",
      "Backend": {
        "Type": "host-gw"
      }
    }
```
- 应用配置：`kubectl apply -f kube-flannel.yaml`
- 删除配置：`kubectl delete -f kube-flannel.yaml`

2. yaml脚本启动pod带参数