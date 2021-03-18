## Deployment概念
- 用于部署无状态的服务，这个最常用的控制器，一般用于管理维护企业内部无状态的微服务，比如 configserver,zuul,springboot等，他可以管理多个副本的pod实现无缝迁移，自动扩缩容，自动灾难恢复，一键回滚等功能。

## Deployment创建
- 通过kubectl命令
- 通过yaml文件


- 手动创建deployment：

```bash
# 通过kubectl命令创建：
kubectl create deployment nginx --image=nginx:1.15.2

# 查看创建的deployment
[root@k8s-master01 ~]# kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           3m8s
```

- 通过手动创建的deployment导出yaml文件，删除不需要的字段即可成为一个可编辑的deployment yaml文件：

```bash
# 导出到yaml文件：
kubectl get deployment nginx -o yaml > nginx-deployment.yaml

# 去掉一些无用字段后：
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2021-03-17T06:58:13Z"
  generation: 1
  labels:
    app: nginx
  name: nginx
  namespace: default
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.15.2
        imagePullPolicy: IfNotPresent
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30

```

## Deployment更新
- kubernetes中只有修改了spec下的template中的内容，才会触发更新生成一个新的rs，否则只在原有rs基础上进行操作。
