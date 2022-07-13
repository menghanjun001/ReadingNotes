[toc]

# 云原生

## 云原生的特点

- 微服务

  非单体应用，拆分为微服务

- 健康报告

  如borg/kubectl下运行的任务都内置了http服务器，用于检测应用现在是否健康——即是否准备好去使用

- 遥测数据

  收集应用的量化指标，如

  - 请求率，收到了多少个请求
  - 错误，应用有多少个错误
  - 时间，应用多久回复

- 弹性

  - 为失败而设计，分布式系统往往不可靠
  - 优雅降级，当高负载情况下，要保证应用总能回复response，即使应用挂掉也要有保底的缓存或降级服务

- 声明式，非命令式

  - 命令式，输入一行shell则输出该条指令的结果，即使应用死亡也有可能输出正确结果
  - 声明式，区别于shell的脚本，声明式信任这样一个方式：给期望结果一个声明，应用就一定能完成

## 云原生的概念

![img](https://1600098707-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LxzmfbtcTYE4On5ZpZ2%2F-LxzmxkNiaJCnAg1d53i%2F-LxznAZVBjiuMaFNlAmN%2Fcloud-native-architecutre-mindnode.jpg?generation=1578397180286626&alt=media)

## 云原生的12 factors+3

1. 基准代码：代码用git管理，打包为docker image放到repository统一管理

2. 依赖：使用包管理工具，如java的gradle maven，go的glide

3. 配置：配置与应用分离

4. 后端服务：计算存储分离降低耦合，如mysql

5. 构建/发布/运行：docker

6. 进程：应用作为无状态的进程，重启后还能恢复到原状态，如不存储session

7. 端口绑定：pod有独立ip，端口不会冲突

8. 并发：容器都是一个进程，可以增加容器的replica实现并发

9. 易处理：快速启动和优雅终止

10. 开发环境和线上环境等价：namespace的资源隔离保证了镜像的使用下， 环境可以轻易复制一套

11. 日志：日志作为事件流使用stout输出，如输出到es保存

12. 管理进程：如kubectl进入到容器内部操作

13. api优先：团队协作，服务间的使用要声明api的规约

14. 监控与告警：应用的性能/健康/日志的监控

15. 认证授权：在应用的上层就做，与服务解耦

    ![img](https://1600098707-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LxzmfbtcTYE4On5ZpZ2%2F-LxzmxkNiaJCnAg1d53i%2F-LxznAZXtvbffvJeF6XG%2F12-factor-app.png?generation=1578397167178564&alt=media)

    

## 云原生与kubernetes

### k8s的资源对象

在k8s中，kubernetes对象是一个重要概念。一个对象描述了几件事：

- 什么应用在容器化部署运行，运行在哪个node上
- 资源的描述
- 应用的重启策略/升级策略/容错策略

总体来说，k8s对象是一个声明式的描述，描述了目的——对象被创建后，就会按照声明的spec所运行。这就是k8s的**期望状态**，也即声明式描述

### 资源配额与限制

- pod级别，最小的资源调度单位
- namespace级别，限制资源隔离的配额

### CI/CD

![img](https://1600098707-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LxzmfbtcTYE4On5ZpZ2%2F-LxzmxkNiaJCnAg1d53i%2F-LxznAZc9r1VcAhWl-zg%2Fkubernetes-jenkins-ci-cd.png?generation=1578397168122934&alt=media)

1. 用户提交的代码内含dockerfile
2. push到gitlab
3. 发布应用的时候，填写git repo/branch/service name/实例个数/资源数量等信息，触发jenkins脚本
4. jenkins ci流水线编译打包docker image，push到repository
5. jenkins ci流水线应用脚本，替换里面的kubernetes的yaml模板的入参
6. 生成应用专属的yaml配置
7. jenkins更新ingress的配置，在ingress代表的lb中加入一条路由信息
8. Jenkins更新dns记录，插入一条边缘节点的ip记录
9. jenkins调用kubernetes的api，部署一个应用

总结就是用户提交到gitlab，gitlab触发jenkins去将应用编译打包（ci），之后jenkins更新lb与dns记录，并触发k8s的api完成应用的部署（cd）。



### devops

如何践行一个devops？笔者归纳：

1. 根据环境（开发/测试/生产）划分namespace，也可以根据项目划分
2. 根据用户划分namespace
3. 封装kubectl，比如加上一个用户身份校验
4. 管理员可以通过dashboard查看不同namespace的状态
5. 所有应用的日志推送到es
6. 通过grafana可以查看所有namespace中应用的状态,即集群监控

原文为[k8s handbook:devops行动指南](https://hezhiqiang.gitbook.io/kubernetes-handbook/yun-yuan-sheng/kubernetes-and-cloud-native-app-overview)



#### 附：cicd和devops是指什么

- cicd
  - continuous integration：持续集成，指coding完项目，pipeline编译打包的过程
  - continuous development：持续部署，指pipeline发布部署的过程
- devops：指开发与运维紧密贴合的部分



## 如何迁移到云原生架构

![img](https://1600098707-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LxzmfbtcTYE4On5ZpZ2%2F-LxzmxkNiaJCnAg1d53i%2F-LxznAZieweS5sVCwA8X%2Fmigrating-hadoop-yarn-to-kubernetes.png?generation=1578397176711103&alt=media)

1. 将原有应用拆解为服务
2. 定义服务的api通信文档
3. 编写启动脚本作为容器的入口
4. 拆解出配置文件
5. 制作容器镜像

详情可见[迁移到云原生应用架构](https://lib.jimmysong.io/migrating-to-cloud-native-application-architectures/)



## service mesh

个人理解是解耦与原应用，将日志/服务发现等服务解耦其实就是service mesh，常见的sidecar部署就属于service mesh。

![img](https://1600098707-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LxzmfbtcTYE4On5ZpZ2%2F-LxzmxkNiaJCnAg1d53i%2F-LxznAZkwUjSIsKoH6Bj%2Fserivce-mesh-control-plane.png?generation=1578397178048201&alt=media)





# k8s概念与原理

## kubernetes架构

### borg架构

borg是kubernetes前身，kubernetes即谷歌的borg的开源版本。

![img](https://1600098707-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LxzmfbtcTYE4On5ZpZ2%2F-LxzmxkNiaJCnAg1d53i%2F-Lxzn3jzSGs_HPAPSLNP%2Fborg.png?generation=1578397164350667&alt=media)



- Borg master：集群的大脑，维护集群状态，将数据持久化
- scheduler：调度器，将应用调度到容器上
- borglet：负责运行容器
- borgcfg：borg的命令行工具，一般通过配置文件来提交任务

### 整体架构

![img](https://1600098707-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LxzmfbtcTYE4On5ZpZ2%2F-LxzmxkNiaJCnAg1d53i%2F-Lxzn3k0WcqIKjDB3g3I%2Farchitecture.png?generation=1578397165138264&alt=media)

![img](https://1600098707-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LxzmfbtcTYE4On5ZpZ2%2F-LxzmxkNiaJCnAg1d53i%2F-Lxzn3k242pwmxarE0U-%2Fkubernetes-high-level-component-archtecture.jpg?generation=1578397164662526&alt=media)

- master：负责维护集群状态
  - etcd：保存集群状态
  - api server：网关，提供认证 鉴权的功能
  - scheduler：调度器，负责资源的调度，按照预定的调度策略调度pod
  - controller manager：负责维护集群的状态，比如故障检测/滚动更新
- node
  - kubelet：负责维护容器的生命周期，负责网络和container runtime
  - container runtime：负责镜像管理和容器的真正运行，由kubelet来管理
  - kube-proxy：为service提供集群内的服务发现和负载均衡
  - pod：多个container，运行服务



### 抽象架构

![img](https://1600098707-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LxzmfbtcTYE4On5ZpZ2%2F-LxzmxkNiaJCnAg1d53i%2F-Lxzn3k4OR0LWhmAwD_7%2Fkubernetes-whole-arch.png?generation=1578397159830067&alt=media)

#### master架构

![img](https://1600098707-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LxzmfbtcTYE4On5ZpZ2%2F-LxzmxkNiaJCnAg1d53i%2F-Lxzn3k6rRPed_XK0fr1%2Fkubernetes-master-arch.png?generation=1578397164272914&alt=media)

#### node架构

![img](https://1600098707-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LxzmfbtcTYE4On5ZpZ2%2F-LxzmxkNiaJCnAg1d53i%2F-Lxzn3k8Fq2fddWo0s5V%2Fkubernetes-node-arch.png?generation=1578397163874162&alt=media)



## kubernetes核心概念和api对象

### api对象

每个api对象都有三大属性：metadata/spec/status

- metadata：元数据，用于标识api对象的，每个对象至少有三个元数据
  - namespace：资源隔离
  - uid：唯一标识
  - name
  - labels（可选）：比如labels设置一个env，通过env=test/prod来区分测试和生产环境
- spec：规约，用于描述api对象的desired state（期望状态），这体现了k8s是属于声明式系统。
- status：当前状态
- kind：用于针对不同的业务类型来决定一个api对象采用什么样的kind

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```



### 业务类型

kubernetes下运行的业务主要分四种类型，这四种类型决定一个api对象该是什么kind

- long running：一个长期运行的服务，如一个web应用
- batch：批处理类型，和long running的区别就是batch属于有头有尾，long running没有尾
- node-daemon：后台支撑类型。比如有的node上需要运行多个相同业务的pod，现在需要在每个node上都至少拥有一个pod提供支撑服务，这个支撑服务可以是db/cache/log/monitor，此类业务称为后台支撑类型
- stateful application：比如一个无状态应用当pod挂掉之后，一个新的pod会被新建。此时期望状态的pod数目才是需要关注的，pod名字是不需要被关注的。但是一个有状态的应用是通过pod名字来跟状态关联的，如果pod挂掉，新建的pod会沿用这个事先确定的pod名。

### workloads 工作负载

workloads代表一组管理pod的资源，这代表无需一个个管理pod，controller会确保当前一组pod会达到desired state。这一张可参考[kubernetes docs:workloads](https://kubernetes.io/docs/concepts/workloads/)。

#### pods

api对象中pod对象很重要，他是资源调度的最小单位，内置很多container来组成一个微服务对外提供服务。**多个container在一个pod中共享网络和filesystem**，通过进程间通信和文件系统组合来完成高效的协作。

#### workload resources

kubernetes内置了几种workload resources，以匹配不同类型的应用：

- Deployment and replica set，这种workload常用于long running的业务：

  - replication controller：早期保证高可用的api对象，目的是保证pod达到desired state

  - replica set：取代了replication controller，一般不单独使用

  - deployment：deployment是更高级的概念。比如一个滚动更新，实际上是新建一个replica set将replica更新到指定数量，然后减少旧replica set中replica的数量，这种行为用一个deployment来统称描述。官方建议使用deployment而不是replica set。通常用法是在spec中指定replicas数量，如下是示例。

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      labels:
        app: nginx
    spec:
      replicas: 3
    ```

    

- Job and cron job，这种workload常用于batch类型的业务，job会创建一个或多个pod，直到指定pod的任务完成后终止

  - job
    - 非并行：指定一个pod，任务完成则终止
    - 指定计数：yaml中描述.spec.completions为非负整数，成功的pod数达到这个指标则终止
    - 工作队列：yaml中描述.spec.parallelism并行运行
  - cron job：定时重复调度的job，所有cron job的调度都是基于controller的时区

  ```yaml
  apiVersion: batch/v1
  kind: Job
  ```

- daemon set，这种workload常用于日志/存储/监控，每个node中必须内置这样一个pod来运行相关的业务。

  ```yaml
  apiVersion: apps/v1
  kind: DaemonSet
  ```

- Stateful set，这种workload常用于一个有状态的应用，管理的pod往往名字有特殊含义，并且区别于replica的pod不会mount自己的volume；stateful set管理的pod会mount自己的volume，以便pod漂移后会重新mount后绑定状态。下面示例一个nginx作为由状态的应用：

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx
    labels:
      app: nginx
  spec:
    ports:
    - port: 80
      name: web
    clusterIP: None
    selector:
      app: nginx
  ```

  ```yaml
  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: web
  spec:
    selector:
      matchLabels:
        app: nginx # has to match .spec.template.metadata.labels
    serviceName: "nginx"
    replicas: 3 # by default is 1
    minReadySeconds: 10 # by default is 0
    template:
      metadata:
        labels:
          app: nginx # has to match .spec.selector.matchLabels
      spec:
        terminationGracePeriodSeconds: 10
        containers:
        - name: nginx
          image: k8s.gcr.io/nginx-slim:0.8
          ports:
          - containerPort: 80
            name: web
          volumeMounts:
          - name: www
            mountPath: /usr/share/nginx/html
    volumeClaimTemplates: # provide stable storage using PersistentVolumes provisioned by a PersistentVolume Provisioner.
    - metadata:
        name: www
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "my-storage-class"
        resources:
          requests:
            storage: 1Gi
  ```





### federation

在云计算概念下，服务的作用范围由近到远分以下几种：

- host/node：同主机
- zone：同zone 跨主机，如xx机房
- region：跨可用区同地区，如上海
- cloud service provider：同供应商跨地区，如上海阿里云/杭州阿里云
- 跨云平台：如金山云/华为云



kubernetes设计就是为了在同zone下使用的，在同zone甚至region下的网络性能才能保证提供高可靠的服务。federation就是作用于跨region（如上海杭州）和跨云平台提供商（金山云和华为云）提出的。



在抽象架构上，federation对标master，包括以下几点：

- 分布式存储，对标单一集群下master的etcd
- api server，提供了网关，认证和鉴权
- controller，维护集群状态，保证cluster的可用



在一个客户端发来请求的时候，在federation首先会做lb，将请求发到合适的cluster下。

详细可参考[kuberfed的定义](https://github.com/kubernetes-sigs/kubefed/blob/master/docs/concepts.md)和



### persistent volume和persistent volume claim

pv和pvc让一个cluster有了独立的存储能力。pv之于pvc和node之于pod的关系是很相似的：

- 资源的提供
  - pv：提供存储资源
  - node：提供cpu和内存
- 资源的使用
  - pvc：使用存储资源
  - pod：使用cpu和内存资源

举例一个pvc，将pod声明为一个volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

### node

node可以看成是一个slave节点，是承载pod运行的主机，可是物理机或是虚拟机。工作主机的特征就是有个kubelet维护容器的生命周期。



### secret

避免把密码明文写在配置文件里。





### interface

在kubernetes中接口主要分三种

- Container runtime interface（CRI）：分管计算资源
- Container network interface（CNI）：分管网络资源
- Container storage interface（CSI）：分管存储资源

#### CRI

CRI主要定义了kubelet与**容器运行时**通信的接口，采用的grpc协议，cri可以被定义为一个service类型的proto，文件详见[github kubernetes cri api](https://github.com/kubernetes/cri-api/blob/c75ef5b/pkg/apis/runtime/v1/api.proto)

什么是容器运行时？

> 负责运行容器的软件

![img](https://1600098707-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LxzmfbtcTYE4On5ZpZ2%2F-LxzmxkNiaJCnAg1d53i%2F-Lxzn-xbNQ0P0j2amd8F%2Fcri-architecture.png?generation=1578397159706418&alt=media)

CRI作为后端接口，与kubelet内置的grpc client通信。CRI中定义了容器和镜像两个服务，因为容器和镜像的生命周期需彼此隔离。

```protobuf
// Runtime service defines the public APIs for remote container runtimes
service RuntimeService {
    
}

// ImageService defines the public APIs for managing images.
service ImageService {
    
}
```

- runtimeService：管理容器的生命周期
- ImageService：管理镜像的pull push 销毁

#### CNI

CNI用于容器创建的时候分配网络，容器销毁的时候也释放网络资源。CNI的接口定义如下：

```go
type CNI interface {
    AddNetworkList(net *NetworkConfigList, rt *RuntimeConf) (types.Result, error) //添加网络列表
    DelNetworkList(net *NetworkConfigList, rt *RuntimeConf) error //删除网络列表

    AddNetwork(net *NetworkConfig, rt *RuntimeConf) (types.Result, error) //添加网络
    DelNetwork(net *NetworkConfig, rt *RuntimeConf) error //删除网络
}
```

kubernetes是不提供任何网络配置的，所以网络需要新增插件来配置。一个CNI插件必须为一个可执行文件。

#### CSI

CSI的使用太复杂，暂只了解一种用法——新建一个persistent volume对象：

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-manually-created-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  csi:
    driver: com.example.team/csi-driver
    volumeHandle: existingVolumeName
    readOnly: false
```





## kubernetes中的网络

