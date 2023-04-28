# chaos-scheduler
## 开发过程
```shell
# 创建项目
mkdir -p $GOPATH/src/extend-k8s.io/chaos-scheduler

# 初始化项目结构
kubebuilder init

# 生成模板
make

# 创建 pod 控制器
kubebuilder create api --kind Pod --group core --version v1

# 编写代码

# 测试
kubectl run nginx-by-chaos-scheduler --image=nginx:1.18-alpine --overrides='{"spec":{"schedulerName":"chaos-scheduler"}}'
```

## 本地启动
```shell
go run .
```

## KubeSchedulerConfiguration 启动调度器
#### ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-scheduler-custom
  namespace: kube-system
data:
  config.yaml: |
    apiVersion: kubescheduler.config.k8s.io/v1
    kind: KubeSchedulerConfiguration
    leaderElection:
      leaderElect: false
    clientConnection:
      kubeconfig: "/etc/kubernetes/scheduler.conf"
    profiles:
      - schedulerName: custom-scheduler
        plugins:
          score:
            disabled:
              - name: NodeResourcesBalancedAllocation
          bind:
            enabled:
              - name: DefaultBinder
                weight: 1
```
#### Scheduler
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-scheduler-custom
  namespace: kube-system
spec:
  containers:
    - name: kube-scheduler-custom
      image: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.25.1
      command:
        - kube-scheduler
        - --config=/etc/kubernetes/config/config.yaml
        - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
        - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
        - --kubeconfig=/etc/kubernetes/scheduler.conf
        - --leader-elect=false
        - --feature-gates=AllBeta=false
      volumeMounts:
        - name: config
          mountPath: /etc/kubernetes/config
          readOnly: true
        - mountPath: /etc/kubernetes/scheduler.conf
          name: kubeconfig
          readOnly: true
  nodeName: xiaolatiao1
  restartPolicy: Always
  volumes:
    - name: config
      configMap:
        name: kube-scheduler-custom
    - hostPath:
        path: /etc/kubernetes/scheduler.conf
        type: FileOrCreate
      name: kubeconfig
```
#### 测试
```shell
kubectl run nginx-by-custom-scheduler --image=nginx --overrides='{"spec":{"schedulerName":"custom-scheduler"}}'
```

## Description
// TODO(user): An in-depth paragraph about your project and overview of use

## Getting Started
You’ll need a Kubernetes cluster to run against. You can use [KIND](https://sigs.k8s.io/kind) to get a local cluster for testing, or run against a remote cluster.
**Note:** Your controller will automatically use the current context in your kubeconfig file (i.e. whatever cluster `kubectl cluster-info` shows).

### Running on the cluster
1. Install Instances of Custom Resources:

```sh
kubectl apply -f config/samples/
```

2. Build and push your image to the location specified by `IMG`:
	
```sh
make docker-build docker-push IMG=<some-registry>/chaos-scheduler:tag
```
	
3. Deploy the controller to the cluster with the image specified by `IMG`:

```sh
make deploy IMG=<some-registry>/chaos-scheduler:tag
```

### Uninstall CRDs
To delete the CRDs from the cluster:

```sh
make uninstall
```

### Undeploy controller
UnDeploy the controller to the cluster:

```sh
make undeploy
```

## Contributing
// TODO(user): Add detailed information on how you would like others to contribute to this project

### How it works
This project aims to follow the Kubernetes [Operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)

It uses [Controllers](https://kubernetes.io/docs/concepts/architecture/controller/) 
which provides a reconcile function responsible for synchronizing resources untile the desired state is reached on the cluster 

### Test It Out
1. Install the CRDs into the cluster:

```sh
make install
```

2. Run your controller (this will run in the foreground, so switch to a new terminal if you want to leave it running):

```sh
make run
```

**NOTE:** You can also run this in one step by running: `make install run`

### Modifying the API definitions
If you are editing the API definitions, generate the manifests such as CRs or CRDs using:

```sh
make manifests
```

**NOTE:** Run `make --help` for more information on all potential `make` targets

More information can be found via the [Kubebuilder Documentation](https://book.kubebuilder.io/introduction.html)

## License

Copyright 2023.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

