# Kubernetes

# Serverless框架 ———— 相关概念

Serverless是的核心思想是**用户无须关注支撑应用服务运行的底层主机**。

用户无须关心软件应用运行涉及的底层服务器的状态、资源（比如CPU、内存、磁盘及网络）及数量。软件应用正常运行所需要的计算资源由底层的云计算平台动态提供。

在Serverless架构下，当用户完成应用开发后，软件应用将被部署到指定的运行环境，这个运行环境不再是具体的一台或多台服务器，而是支持Serverless的云计算平台。

有客户端请求到达或特定事件发生时，云计算平台负责将应用部署到某台Serverless云计算平台的主机中。

Serverless云计算平台保证该主机提供应用正常运行所需的计算资源。

在访问量升高时，云计算平台动态地增加应用的部署实例。当应用空闲一段时间后，云计算平台自动将应用从主机中卸载，并回收资源。

用户之所以不用再关注服务器是因为底层的云计算平台完成了大量的自动化工作。

## FaaS与BaaS

目前业界的各类Serverless实现按功能而言，主要为应用服务提供了两个方面的支持：函数即服务（Function as a Service，FaaS）以及后台即服务（Backend as a Service，BaaS）。

- FaaS

FaaS提供了一个计算平台，在这个平台上，应用以一个或多个函数的形式开发、运行和管理等。

FaaS可以根据实际的访问量进行应用的自动化动态加载和资源的自动化动态分配。大多数FaaS平台可以根据预定义的事件触发指定的函数应用逻辑。

FaaS是目前Serverless架构实现的一个重要手段。FaaS平台的特点在很大程度上影响了目前Serverless应用的架构和实现方式，FaaS平台是一个完整的Serverless实现的重要组成部分，仅仅通过FaaS平台并不能完全实现Serverless架构的落地。

FaaS为应用的开发、运行和管理提供良好的Serverless基础。但是对应用整体系统而言，并没有完全实现Serverless化。因此，在实现应用架构Serverless化时，也应该实现应用所依赖的服务的Serverless化。

- BaaS

为了实现应用后台服务的Serverless化，BaaS（后台即服务）也应该被纳入一个完整的Serverless实现的范畴内。

BaaS涵盖的范围很广泛，包含任何应用所依赖的服务。AWS的DynamoDB数据库就是DBaaS的一个例子。用户按需申请使用数据库服务，而无须关注数据库的运维和缩扩容。数据库的运维完全由服务提供方AWS负责。

要实现完整的Serverless架构，用户必须结合FaaS和BaaS的功能，使得应用整体的系统架构实现Serverless化，最大化地实现Serverless架构所承诺带来的价值。

因此，一个完整的Serverless实现，必须同时提供FaaS和BaaS两个方面的功能。

## Serverless架构下的应用与传统架构下的应用的异同

- 传统架构
<div align="center"> <img src="https://github.com/Liccol/Liccol.github.io/blob/master/img/kubernetes-1.webp" width=""> </div><br>

- Serverless应用架构
<div align="center"> <img src="https://upload-images.jianshu.io/upload_images/3342415-54878079b8c5750b.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp" width=""> </div><br>

Serverless架构下的应用架构图。这个应用实现的功能和前文的应用一样，即为用户提供订单的增删查改服务。应用被部署在Serverless平台之上。应用的功能点变成若干个函数定义，部署于FaaS中。

- 两种架构的比较

传统架构的应用部署在主机之上，而Serverless架构的应用部署于Serverless平台之上，由Serverless平台提供运行所需的计算资源传统架构的应用里，所有的逻辑都集中在同一个部署交付件中。

Serverless应用的逻辑层部署运行在Serverless平台的FaaS服务之上。因此，应用的逻辑被打散成多个独立的细颗粒度的函数逻辑。

因为业务逻辑运行在FaaS服务之上，所以相关的业务逻辑拥有事件驱动、资源自动弹性扩展、高可用等特性。

用户也无须运维业务逻辑所消耗的计算资源，Serverless架构的应用所依赖的数据库从具体的特定数据库实例，变成了数据库服务。用户无须关注数据库所消耗的计算资源的运维。

在Serverless架构下，由于应用的逻辑分散成了若干个函数，推荐通过API网关对这些函数逻辑进行统一的管控（如流量控制、安全管控、版本管理等）。

在完备的Serverless平台上，API网关也会作为一种用户可以快速消费的服务而存在。在Serverless架构下，具体的主机资源不再是用户关注的对象，不存于应用架构图中。取而代之的是Serverless平台及其子服务，如FaaS和各类BaaS服务。

## Serveless技术特点

- 按需加载

在Serverless架构下，应用的加载（load）和卸载（unload）由Serverless云计算平台控制。这意味着应用不总是一直在线的。只有当有请求到达或者有事件发生时才会被部署和启动。

- 事件驱动

Serverless架构的应用并不总是一直在线，而是按需加载执行。

通过将不同事件来源（Event Source）的事件（Event）与特定的函数进行关联，实现对不同事件采取不同的反应动作，这样可以非常容易地实现事件驱动（Event Driven）架构。

- 状态非本地持久化

应用不再与特定的服务器关联。因此应用的状态不能，也不会保存在其运行的服务器之上，

- 非会话保持

应用不再与特定的服务器关联。每次处理请求的应用实例可能是相同服务器上的应用实例，也可能是新生成的服务器上的应用实例。因此，Serverless架构更适合无状态的应用。

- 自动弹性伸缩

Serverless应用原生可以支持高可用，可以应对突发的高访问量。应用实例数量根据实际的访问量由云计算平台进行弹性的自动扩展或收缩，云

- 应用函数化

Serverless架构下的应用会被函数化，但不能说Serverless就是Function as a Service（FaaS）。

- 依赖服务化

在Server-less架构下的应用的依赖应该服务化和无服务器化，也就是实现BaaS。

所有应用依赖的服务都是一个个Backend Service，应用通过BaaS方便获取，而无须关心底层细节。

和FaaS一样，BaaS是Serverless架构实现的另一个重要组件。

## 缺陷

- 控制力

对于一些希望掌控底层计算资源的应用场景，并不是最合适的选择。

- 可移植性

Serverless应用的实现在很大程度上依赖于Serverless平台及该平台上的FaaS和BaaS服务。不同IT厂商的Serverless平台和解决方案的具体实现并不相同。

- 安全性

Serverless架构下，用户不能直接控制应用实际所运行的主机。在运行时可能共用底层的主机资源。对于一些安全性要求较高的应用，这将带来潜在的安全风险

- 性能

应用的首次加载及重新加载的过程将产生一定的延时。对于一些对延时敏感的应用，需要通过预先加载或延长空闲超时时间等手段进行处理。

- 执行时长

大部分Serverless平台对FaaS函数的执行时长存在限制。因此Serverless应用**更适合一些执行时长较短的作业**。

- 技术成熟度

目前Serverless相关平台、工具和框架还处在一个不断变化和演进的阶段，开发和调试的用户体验还需要进一步提升。Serverless相关的文档和资料相对比较少，深入了解Serverless架构的架构师、开发人员和运维人员也相对较少

## 相关技术

按所管控的计算资源的范围来划分，云计算模式可以分为基础架构即服务（Infrastructure as a Service）、平台即服务（Platform as a Service，PaaS）以及软件即服务（Software as a Service，SaaS）。

### IaaS、PaaS与SaaS
- IaaS(购买云存储服务器)

应用架构底层的网络、存储和计算资源（主机、物理机或虚拟机），由云平台供应商提供和运维。

用户在云平台上付费申请所需的网络、存储和计算资源，云平台供应商在一定时间内提供。

- PaaS(为服务器配置好OS等，可以直接在上面开发软件)

提供了应用的运行环境（如应用运行时）、应用依赖的服务（如数据库、中间件、负载均衡、构建服务、发布服务）以及底层所需的计算资源，用户可以把精力集中在应用的开发和创新上。

- SaaS(购买云上的服务，如搭好的网页、软件等)

用户完全不用管理任何应用和基础设施，从而变成云服务的消费者，直接消费已经准备好的应用。

### Serverless与容器

容器架构中最小的运行单元是容器，而Serverless中则是函数。

容器应用一般是预先部署，然后持续在线。这看起来和Serverless不大一样，因为FaaS是按需加载和执行的。

而在Serverless架构中，应用是按需加载和执行的。这意味着理论上Serverless的资源使用效率更高。


但其实是可以的，虽然目前容器内一般运行的是一个完整的应用，但是将容器内运行的对象变成函数显然无困难。

Kubernetes上默认没有事件触发的支持，无法做到按需部署容器应用。

但是通过Kubernetes叠加上一些FaaS框架运行包含函数逻辑的容器，用户很容易使Kubernetes具备FaaS服务的能力。

所以也就是说，可以通过Kubernetes+事件触发容器来实现Serverless的功能。

### Serverless技术基础———— FaaS

每一个函数完成一个相对简单的业务逻辑，一个完整的应用由若干个函数组成。因为FaaS和Serverless的关系密切，因此FaaS的特点同时也是Serverless平台的特点：

- 抽象了底层计算资源
- 按使用量付费
- 自动弹性扩展
- 事件驱动

宏观来看，一个FaaS平台的架构中包含如下主要组件：

- 函数定义（Function Definition）：一个函数实现一个业务逻辑
- 函数实例（Function Instance）：在运行状态的应用函数的实例
- 控制器（Controller）：负责应用函数的加载、执行等流程的管理
- 事件（Event）：事件驱动架构中的事件
- 事件源（Event Source）：事件驱动架构中的事件来源。可以是一个数据库中插入了新的记录，也可以是一个目录里删除了一个文件，或者是消息队列收到了新的消息
- 触发规则（Trigger Rule）：定义事件与函数的关系及触发的规则
- 平台服务（Platform Service）：支撑应用运行的各类底层服务，如计算资源、数据存储等

函数的生命周期：在FaaS上一个函数从创建到执行的生命周期
- 用户根据所选定的FaaS平台的规范进行函数应用的开发。
- 编写好的函数将上传至FaaS平台。平台将负责编译和构建这些函数，并将构建的输出保存。
- 用户设置函数被触发的规则，将事件源与特定版本的函数进行关联。
- 当事件到达且满足触发规则时，平台将会部署、编译构建后的函数并执行。平台将监控函数执行的状态，根据请求量的大小，平台负责对函数实例进行扩容和缩容

函数工作流

当涉及多个函数执行时，就需要有逻辑处理执行的顺序、错误重试、异常捕获以及状态传递等细节。

一些FaaS实现开始提供针对FaaS函数的流程编排服务或工具，以简化FaaS应用的流程编排，如AWS Step Functions和Fission Workflows。

### BaaS

BaaS的价值

后端即服务（Backend as a Service，BaaS）

通过BaaS平台，用户的应用程序可以对接后端的各种服务，省去了用户学习各种技术和中间件的成本，降低了应用开发的复杂度。BaaS的服务往往由服务供应商提供，用户无须关心底层细节，无须维护相关资源

# Serverless 产品 ———— kubeless

主要讲一下k8s的serverless产品——kubeless。

kubeless 是 kubernetes native 的无服务计算框架，它可以让用户在 kubernetes 之上使用 FaaS 构建高级应用程序。

Kubless 有三个核心概念：

- **Functions**：需要被执行的用户代码，同时包含运行时依赖、构建指令等信息；
- **Triggers**：和函数关联的事件源(通过设置trigger快速触发后台函数运行)。如果把事件源比作生产者，函数比作执行者，那么触发器就是联系两者的桥梁；
- **Runtime**：代表函数运行时所依赖的环境。

## 原理

介绍是怎么做到产品提供的基本功能的：

- 敏捷构建：基于用户提交的源码迅速构建可执行的函数，简化部署流程；
- 灵活触发：基于各类事件触发函数的执行，方便快捷地集成新的事件源；
- 自动伸缩：根据业务需求，自动完成扩容缩容，无须人工干预。

### 敏捷构建

<div align="center"> <img src="https://cdn.nlark.com/lark/0/2018/png/86722/1543736647858-7fe371aa-2f30-4e4b-bdb7-d90007aae53a.png" width=""> </div><br>

函数生命周期的定义如图。

用户只需提供源码和函数说明，构建部署等工作通常由 serverless 平台完成。 因此，基于用户提交的源码迅速构建可执行函数是 serverless 产品必须具备的基础能力。
```
kubeless function deploy hello --runtime python2.7 \
                                --from-file test.py \
                                --handler test.hello
```

配置时的参数为：

hello：将要部署的函数名称；

--runtime python2.7： 指定使用 python 2.7 作为运行环境。

--from-file test.py：指定函数源码文件（支持 zip 格式）。

--handler test.hello：指定使用 test.py 中的 hello 方法处理请求。(参考对象.方法的写法)

#### 函数资源与K8s Operator

Kubeless 函数是一个自定义 K8s 对象，本质上是 k8s operator。k8s operator 原理如下图所示：

<div align="center"> <img src="https://cdn.nlark.com/lark/0/2018/png/86722/1543562354219-9e3e57ca-5d59-48f0-b228-487eda4fdf35.png" width=""> </div><br>

以 kubeless 函数为例，描述 K8s operator 的一般工作流程：

1. 使用 k8s 的 CustomResourceDefinition(CRD) 定义资源，这里创建了一个名为functions.kubeless.io的 CRD 来代表 kubeless 函数；

2. 创建一个 controller 监听自定义资源的 ADD、UPDATE、DELETE 事件并绑定 hander。这里创建了一个名为function-controller的 CRD controller，该 controller 会监听针对 function 的 ADD、UPDATE、DELETE 事件，并绑定三种 handler（参阅 AddEventHandler）；

3. 用户执行创建、更新、删除自定义资源的命令；

4. Controller 根据监听到的事件调用相应的 handler。

#### 函数构成

Kubeless 的 function-controller监听到针对 function 的 ADD 事件后，会触发相应 handler 创建函数。一个函数由若干 K8s 对象组成，包括 ConfigMap、Service、Deployment、Pod 等，其结构如下图所示：

<div align="center"> <img src="https://cdn.nlark.com/lark/0/2018/png/86722/1543592884194-d679c8cf-c8cf-4ce0-8124-7a574e7d7ae7.png" width=""> </div><br>

这个构造的过程应该是去构建相应的各个yaml文件。
ocnfigmap:
```
apiVersion: v1
data:
  handler: test.hello
  # 函数依赖的第三方 python 库
  requirements.txt: |
    kubernetes==2.0.0
  # 函数源码
  test.py: |
    def hello(event, context):
      print event
      return event['data']
kind: ConfigMap
metadata:
  labels:
    created-by: kubeless
    function: hello
  # 该 ConfigMap 名称
  name: hello
  namespace: default
```

Service指定函数访问方式

```
apiVersion: v1
kind: Service
metadata:
  labels:
    created-by: kubeless
    function: hello
  # 该 Service 名称
  name: hello
  namespace: default
  ...
spec:
  clusterIP: 10.109.2.217
  ports:
  - name: http-function-port
    port: 8080
    protocol: TCP
    targetPort: 8080
  # 选择对应函数
  selector:
    created-by: kubeless
    function: hello
  # Service 类型
  type: ClusterIP
...
```

Deployment 用于编排执行函数逻辑的 Pods，通过它可以描述函数期望的个数。

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    created-by: kubeless
    function: hello
  name: hello
  namespace: default
  ...
spec:
  # 指定函数期望的个数
  replicas: 1
```

Pod 是真正执行函数逻辑的容器。

Pod 中的 volumes 段指定了该函数的 ConfigMap。

这会将 ConfigMap 中的源码和依赖添加到 volumeMounts.mountPath 指定的目录里面。从容器视角来看，文件路径为/src/test.py和 /src/requirements。

```
...
  volumeMounts:
  - mountPath: /kubeless
    name: hello
  - mountPath: /src
    name: hello-deps
volumes:
- emptyDir: {}
  name: hello
- configMap:
    defaultMode: 420
    name: hello
...
```

Init Container

Pod 中的 Init Container 主要作用如下：

- 将源码和依赖文件拷贝到指定目录；
- 安装第三方依赖。

Func Container

Pod 中的 Func Container 会加载 Init Container 准备好的源码和依赖并执行函数。

不同 runtime 加载代码的方式大同小异。主要流程都是解析api获得的json内容，分配给制定好的字段，然后启动。

#### 灵活触发(trigger的多种触发方式)

函数的触发方式分成了如下图所示的几种类别。

<div align="center"> <img src="https://cdn.nlark.com/lark/0/2018/png/86722/1543734315398-14f58052-43b3-45ab-a9b6-5742e6914576.png" width=""> </div><br>
<div align="center"> <img src="https://cdn.nlark.com/lark/0/2018/png/86722/1543390616914-71b0b63f-7c2f-43e8-a7b5-b1aee9d9fb06.png" width=""> </div><br>

触发方式|类别
kubeless CLI|Synchronous Req/Rep
Http Trigger|Synchronous Req/Rep
Cronjob Trigger|Job (Master/Worker)
Kafka Trigger|Async Message Queue
Nats Trigger|Async Message Queue
Kinesis Trigger|Message Stream

##### HTTP trigger

Kubeless 利用 K8s ingress 机制实现了 http trigger。

Kubeless 创建了一个名为httptriggers.kubeless.io的 CRD 来代表 http trigger 对象。

同时，kubeless 包含一个名为http-trigger-controller的 CRD controller，它会持续监听针对 http trigger 和 function 的 ADD、UPDATE、DELETE 事件，并执行对应的操作。

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  # 该 Ingress 的名字，即创建 http trigger 时指定的 name
  name: http-hello
  ...
spec:
  rules:
  - host: example.com
    http:
      paths:
      - backend:
          # 指向 kubeless 为函数 hello 创建的 ClusterIP 类型的 Service
          serviceName: hello
          servicePort: 8080
        path: /echo
```

Ingress 只是用于描述路由规则，要让规则生效、实现请求转发，集群中需要有一个正在运行的 ingress controller。可供选择的 ingress controller 有 Contour、F5 BIG-IP Controller for Kubernetes、Kong Ingress Controllerfor Kubernetes、NGINX Ingress Controller for Kubernetes、Traefik 等。

这种路由规则描述和路由功能实现相分离的思想很好地提现了 K8s 始终坚持的需求和供给分离的设计理念。

上文中的命令在创建 trigger 时指定了 nginx 作为 gateway，因此需要部署一个 nginx-ingress-controller。该 controller 的基本工作原理如下：

- 以 pod 的形式运行在独立的命名空间中；
- 以 hostPort 的形式暴露出来供外界访问；
- 内部运行着一个 nginx 实例；
- 监听和 ingress、service 等资源相关的事件。如果发现这些事件最终会影响到路由规则，ingress controller 会采用向 Lua hander 发送新的 endpoints 列表或者直接修改 nginx.conf 并 reload nginx 等手段达到更新路由规则的目的。

完成上述工作后，我们便可以通过发送 HTTP 请求触发函数 hello 的执行：

- HTTP 请求首先会由 nginx-ingress-controller 中的 nginx 处理；
- Nginx 根据 nginx.conf 中的路由规则将请求转发给函数对应的 service；
- 最后，请求会转发至挂载在 service 后的某个函数进行处理。

##### Cronjob trigger

如果希望**定期**触发函数执行，需要为函数创建 cronjob 触发器。

K8s 支持通过 CronJob 定期运行任务，kubeless 利用这个特性实现了 cronjob trigger。Kubeless 创建了一个名为cronjobtriggers.kubeless.io的 CRD 来代表 cronjob trigger 对象。

同时，kubeless 包含一个名为cronjob-trigger-controller的 CRD controller，它会持续监听针对 cronjob trigger 和 function 的 ADD、UPDATE、DELETE 事件，并执行对应的操作。

使用这个命令可以创建一个每分钟出发hello函数的触发器。
```
kubeless trigger cronjob create scheduled-invoke-hello --function=hello --schedule="*/1 * * * *"
```

##### 自定义

如果发现 kubeless 默认提供的触发器无法满足业务需求，可以自定义新的触发器。新触发器的构建流程如下：

- 为新的事件源创建一个 CRD 来描述事件源触发器；
- 在自定义资源对象的 spec 里描述该事件源的属性，例如 KafkaTriggerSpec、HTTPTriggerSpec；
- 为该 CRD 创建一个 CRD controller。
	- 该 controller 需要持续监听针对事件源触发器和 function 的 CRUD 操作并作出正确的处理。例如，controller 监听到 function 的删除事件，需要把和该 function 关联的触发器一并删掉；
	- 当事件发生时，触发关联函数的执行。
	
我们可以看到，自定义 trigger 的流程遵循了 K8s Operator 设计模式。

### 自动伸缩

K8s 通过 Horizontal Pod Autoscaler 实现 pod 的自动水平伸缩。Kubeless 的 function 通过 K8s deployment 部署运行，因此天然可以利用 HPA 实现自动伸缩。

- 内置度量指标 cpu

CPU 使用率等，对于这类指标 HPA 可以通过 metrics API 从 Metrics Server 中获取数据。Metrics Server 是 Heapster 的继承者，它可以从 Kubelet、cAdvisor 中获取度量数据。

- 自定义度量指标 qps

QPS 属于自定义度量指标，想要获取这类指标的度量数据需要完成下列步骤。

部署用于存储度量数据的系统，这里选择Prometheus。

Prometheus 是一套开源监控&告警&时序数据解决方案。采集度量数据，并写入部署好的 Prometheus 中。

Kubeless 提供的函数框架会在函数每次被调用时，将下列度量数据 function_duration_seconds、function_calls_total、function_failures_total 写入 Prometheus。

部署实现了 custom metrics API 的 custom API server。这里，因为度量数据被存入了 Prometheus，因此选择部署 k8s-prometheus-adapter，它可以从 Prometheus 中获取度量数据。

完成上述步骤后，HPA 就可以通过 custom metrics API 从 Prometheus Adapter 中获取 qps 度量数据。详细配置步骤可参考文章 kubeless-autoscaling。

- 其他度量指标自定义

有时基于 cpu 和 qps 这两种度量指标对函数进行自动伸缩还远远不够。如果希望基于其它度量指标，需要了解 K8s 定义的度量指标类型及其获取方式。

目前，K8s 1.13 版本支持的度量指标类型如下：

类型|简介|获取方式
Object|代表 k8s 对象的度量指标，例如上文提到的 Service 对象的 function_calls 指标。|custom.metrics.k8s.io
度量数据采集后，需要通过已有适配器或自己实现适配器获取数据。目前已有的适配器包括 k8s-prometheus-adapter、azure-k8s-metrics-adapter、k8s-stackdriver 等。
Pod|代表伸缩目标中每个 pod 的自定义度量指标，例如 pod 每秒处理的事务数。在与目标值比较前需要除以 pod 个数。|同上
Resource|代表伸缩目标中每个 pod 的 K8s 内置资源指标（如 CPU、Memory）。在与目标值比较前需要除以 pod 个数。|metrics.k8s.io
从 Metrics Server 或 Heapster 中获取度量数据。
External|External 是一个全局度量指标，它与任何 K8s 对象无关。它允许伸缩目标基于来自 cluster 外部的信息进行伸缩（如外部负载均衡器的 QPS，云消息服务中的队列长度）。|external.metrics.k8s.io
度量数据采集后，需要云平台厂商提供适配器或自己实现。目前已有的适配器包括 azure-k8s-metrics-adapter、k8s-stackdriver 等。

准备好相应的度量数据和获取数据的组件，HPA 就能基于它们对函数进行自动伸缩。

#### 度量数据使用

- 基于CPU使用率

为该函数创建一个基于 cpu 使用率的 HPA，它将运行该函数的 pod 数量控制在 1 到 3 之间，并通过增加或减少 pod 个数使得所有 pod 的平均 cpu 使用率维持在 70%。

通过--参数的形式把目标传递进去

```
kubeless autoscale create hello --metric=cpu --min=1 --max=3 --value=70
```

- 基于 qps

以下命令将为函数 hello 创建一个基于 qps 的 HPA，它将运行该函数的 pod 数量控制在 1 到 5 之间，并通过增加或减少 pod 个数确保所有挂在服务 hello 后的 pod 每秒能处理的请求次数之和达到 2000。
```
kubeless autoscale create hello --metric=qps --min=1 --max=5 --value=2k
```

- 基于多项指标

这种需要通过yaml创建一个指定的HPA控制器。

```
kind: HorizontalPodAutoscaler
apiVersion: autoscaling/v2alpha1
metadata:
  name: hello-cpu-and-memory
  namespace: default
  labels:
    created-by: kubeless
    function: hello
spec:
  scaleTargetRef:
    kind: Deployment
    name: hello
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 50
  - type: Resource
    resource:
      name: memory
      targetAverageValue: 200Mi
```

### 自动伸缩策略
一个理想的自动伸缩策略应当处理好下列场景：

- 当负载激增时，函数能迅速扩展以应对突发流量；
- 当负载下降时，函数能立即收缩以节省资源消耗；
- 具备抗噪声干扰能力，能够精确计算出目标容量；
- 能够避免自动伸缩过于频繁造成系统抖动。

Kubeless 依赖的 HPA 充分考虑了上述情形，不断改进和完善其使用的自动伸缩策略。下面以 K8s 1.13 版本为例描述该策略。

HPA 每隔一段时间会根据获取的度量数据同步一次和该 HPA 关联的 pod 个数。

时间间隔通过 kube-controller-manager 的参数--horizontal-pod-autoscaler-sync-period指定，默认为 15s。

在每一次同步过程中，HPA 需要经历如下图所示的计算流程。

<div align="center"> <img src="https://cdn.nlark.com/lark/0/2018/png/86722/1543237287466-eacc4930-922f-4e21-9eb3-60c29da4397f.png" width=""> </div><br>

#### 计算目标副本数

计算 HPA 列表中每项指标需要的 pod 数量中的最大值作为目标。在计算每项指标的 replicaCountProposal 过程中会考虑下列因素：

允许目标度量值和实际度量值存在一定程度的误差，如果在误差范围内直接使用 currentReplicas 作为 replicaCountProposal。

这样做是为了避免伸缩过于频繁造成系统抖动，该误差值可以通过--horizontal-pod-autoscaler-tolerance指定。

##### pod刚启动时的特殊处理

当一个 pod 刚启动时，该 pod 反映的度量值往往不是很准确，HPA 会将这种 pod 视为 unready。在计算度量值时，HPA 会跳过处于 unready 状态的 pod。这样做是为了消除噪声干扰。

可以通过--horizontal-pod-autoscaler-cpu-initialization-period和--horizontal-pod-autoscaler-initial-readiness-delay调整 pod 被认为处于 unready 状态的时间。

#### 平滑目标副本数
将最近一段时间计算出的 metricDesiredReplicas 记录下来，取其中的最大值作为 stabilizedRecommendation。

这样做是为了让缩容过程变得平滑，消除度量数据异常波动造成的影响。(比如目标副本数一直在小幅波动，那倒不如直接取个近期的最大，浪费就浪费了)。

该时间段可以通过参数--horizontal-pod-autoscaler-downscale-stabilization-window指定，默认为 5 分钟。

#### 规范目标副本数

限制 desiredReplicas 最大为 currentReplicas \* scaleUpLimitFactor(等于2)，这样做是为了防止因 采集到了“虚假的”度量数据造成扩容过快。

限制 desiredReplicas 大于等于 hpaMinReplicas，小于等于 hpaMaxReplicas。

#### 执行扩容缩容操作

如果通过上述步骤计算出的 desiredReplicas 不等于 currentReplicas，则“执行”扩容缩容操作。

这里所说的执行只是将 desiredReplicas 赋值给 RC 或 Deployment 中的 replicas，pod 的创建销毁会由 kube-scheduler 和 worker node 上的 kubelet 异步完成的。

## 存在问题

Kubeless 基于 K8s 提供了较为完整的 serverless 解决方案，但和一些商业 serverless 产品还存在一定差距：

- Kubeless 并未在镜像拉取、代码下载、容器启动等方面做过多优化，导致函数冷启动时间过长；
- Kubeless 并未过多考虑多租户的问题，如果希望多个用户的函数运行在同一个集群里，还需要进行二次开发；
- Kubeless 不支持函数间的协调管理，无法像 AWS Step Functions 那样自定义工作流。

# 常用的Serverless触发流程

<div align="center"> <img src="https://upload-images.jianshu.io/upload_images/3342415-11aa91afa043ae7a.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp" width=""> </div><br>

# Kubernetes 内容

- 控制平面Master组件：

apiserver+

controller-manager(kube:节点/副本/端点/服务帐户和令牌控制器和cloud两种)+

scheduler(监视新创建的未指定运行节点的 Pod，并选择节点让 Pod 在上面运行，即Pod的运行调度，亲和性等)+

etcd(保存 Kubernetes 所有集群数据的后台数据库)

- Node组件：

kubelet(每个节点上运行的代理。它保证K8S创建的容器都运行在 Pod 中。接收 PodSpecs，确保这些 PodSpecs 中描述的容器处于运行状态且健康。)

kube-proxy(集群中每个节点上运行的网络代理,实现 Kubernetes Service 概念的一部分。维护节点上的网络规则。这些网络规则允许从集群内部或外部的网络会话与 Pod 进行网络通信。)

容器运行时（Container Runtime。负责运行容器的软件。）

- 插件

插件使用 Kubernetes 资源（DaemonSet、 Deployment等）实现集群功能。 因为这些插件提供集群级别的功能，插件中命名空间域的资源属于 kube-system 命名空间。

DNS

几乎所有 Kubernetes 集群都应该 有集群 DNS， 因为很多示例都需要 DNS 服务。

集群 DNS 是一个 DNS 服务器，和环境中的其他 DNS 服务器一起工作，它为 Kubernetes 服务提供 DNS 记录。

Kubernetes 启动的容器自动将此 DNS 服务器包含在其 DNS 搜索列表中。

DashBoard

通用的、基于 Web 的用户界面。 它使用户可以管理集群中运行的应用程序以及集群本身并进行故障排除。

容器资源监控

metrics相关

集群层面日志

日至存储相关

## K8S对象

Kubernetes 对象 是持久化的实体，通过.yaml创建。 Kubernetes 使用这些实体去表示整个集群的状态。特别地，它们描述了如下信息：

- 哪些容器化应用在运行（以及在哪些节点上），通过matchlabels等字段

- 可以被应用使用的资源，通过request和limit等字段

- 关于应用运行时表现的策略，比如重启策略、升级策略，以及容错策略，如滚动升级等

### yaml必须字段

需要配置如下的字段：

apiVersion - 创建该对象所使用的 Kubernetes API 的版本

kind - 想要创建的对象的类别，deployment/service等

metadata - 帮助**唯一性标识对象的一些数据**，包括一个 name 字符串、UID 和可选的 namespace，如name:test

你也需要提供对象的 **spec** 字段。可以包含label等特定内容，包含replicas指定副本数目，包含template指定模板（包括容器的镜像/端口等，容器metadata如labels等）

### 对象管理

一般都是用
```
kubectl create -f test.yaml//创建
kubectl apply -f test.yaml//更新
kubectl delete -f test.yaml//删除
kubectl diff -f test.yaml//查看更改
```

使用matadata中的内容唯一标识数据，而其他内容需要使用它的时候，会使用这个k-v对的值来选择。

一个 Service 指向的一组 pods 是由标签选择算符定义的。同样，一个 ReplicationController 应该管理的 pods 的数量也是由标签选择算符定义的。

使用选择：
```
selector:
    component: redis
```

此外，metadata中还可以加上注解annotation，来添加一些额外信息
```
apiVersion: v1
kind: Pod
metadata:
  name: annotations-demo
  annotations:
    imageregistry: "https://hub.docker.com/"
```

## 节点Node

向 API 服务器添加节点的方式主要有两种：

1. 节点上的 kubelet 向控制面执行自注册；
2. 你，或者别的什么人，手动添加一个 Node 对象。

### 节点状态

- 地址

HostName：由节点的内核设置。可以通过 kubelet 的 --hostname-override 参数覆盖。
ExternalIP：通常是节点的可外部路由（从集群外可访问）的 IP 地址。
InternalIP：通常是节点的仅可在集群内部路由的 IP 地址。

- 状况

```
"conditions": [
  {
    "type": "Ready",
    "status": "True",
    "reason": "KubeletReady",
    "message": "kubelet is posting ready status",
    "lastHeartbeatTime": "2019-06-05T18:38:35Z",
    "lastTransitionTime": "2019-06-05T11:41:27Z"
  }
]
```

### 控制面到节点通信

Kubernetes 采用的是中心辐射型（Hub-and-Spoke）API 模式。 

所有从集群（或所运行的 Pods）发出的 API 调用都终止于 apiserver（其它控制面组件都没有被设计为可暴露远程服务）。 

apiserver 被配置为在一个安全的 HTTPS 端口（443）上监听远程连接请求， 并启用一种或多种形式的客户端身份认证机制。 

### 控制面到节点

有两种主要的通信路径。 第一种是从 apiserver 到集群中每个节点上运行的 kubelet 进程。 第二种是从 apiserver 通过它的代理功能连接到任何节点、Pod 或者服务。

- API 服务器到 kubelet

从 apiserver 到 kubelet 的连接用于：

1. 获取 Pod 日志
2. 挂接（通过 kubectl）到运行中的 Pod
3. 提供 kubelet 的端口转发功能。

这些连接终止于 kubelet 的 HTTPS 末端。 默认情况下，apiserver 不检查 kubelet 的服务证书。这使得此类连接容易受到中间人攻击， 在非受信网络或公开网络上运行也是 不安全的。

为了对这个连接进行认证，使用 --kubelet-certificate-authority 标志给 apiserver 提供一个根证书包，用于 kubelet 的服务证书。

如果无法实现这点，又要求避免在非受信网络或公共网络上进行连接，可在 apiserver 和 kubelet 之间使用 SSH 隧道。

最后，应该启用 Kubelet 用户认证和/或鉴权 来保护 kubelet API。

- apiserver 到节点、Pod 和服务

从 apiserver 到节点、Pod 或服务的连接默认为纯 HTTP 方式，因此既没有认证，也没有加密。 

这些连接可通过给 API URL 中的节点、Pod 或服务名称添加前缀 https: 来运行在安全的 HTTPS 连接上。 

不过这些连接既不会验证 HTTPS 末端提供的证书，也不会提供客户端证书。 

因此，虽然连接是加密的，仍无法提供任何完整性保证。 

这些连接 目前还不能安全地 在非受信网络或公共网络上运行。

### Pods

Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。

Pod 的共享上下文包括一组 Linux namespace、控制组（cgroup）和一些其他的用来隔离 Docker 容器的技术。 在 Pod 的上下文中，每个独立的应用可能会进一步实施隔离。

就 Docker 概念的术语而言，Pod 类似于共享名字空间和文件系统卷的一组 Docker 容器。

#### pause容器

Pod = Pause容器 + 其他业务容器。

每个Pod里运行着一个特殊的被称之为Pause的容器，其他容器则为业务容器，这些**业务容器共享Pause容器的网络栈和Volume挂载卷**

pause容器，全称infrastructure container（又叫infra）基础容器

pause容器的PID是1。kubernetes中的**pause容器主要为每个业务容器提供以下功能，为其他业务容器提供各类namespace资源**：

PID命名空间：Pod中的不同应用程序可以看到其他应用程序的进程ID。

网络命名空间：Pod中的多个容器能够访问同一个IP和端口范围。

IPC命名空间：Pod中的多个容器能够使用SystemV IPC或POSIX消息队列进行通信。

UTS命名空间：Pod中的多个容器共享一个主机名；Volumes（共享存储卷）：

Pod中的各个容器可以访问在Pod级别定义的Volumes。

#### 静态Pod

仅存在特定Node上的Pod，无法通过api server控制，无法与RC/deployment/daemonset关联，kubectl无法对其进行健康检查，由kubectl创建，总在kubectl所在的Node上运行。

主要是把kubectl的启动参数--config=yaml存放位置，重启kubectl，就可以创建静态pod。或者通过--manifest-url=目标yaml网址来创建，kubectl定期扫描两种方式的位置，有内容就创建静态pod。

只能通过把对应位置的内容rm掉才可以删除，否则直接kubectl删除的时候会变成pending状态，等待中，不会被删除。

#### 共享卷

pod中的容器共享本地存储卷，container1和container2都可以使用。

```
volumeMounts:
- name: logs
  mountPath: /usr/local/log
```

#### 生命周期和重启策略

pending|镜像仍未创建或在下载中
running|至少有一个容器运行/正在启动/正在重启
succeeded|所有容器执行成功后退出，不再重启
failed|至少有一个容器退出为失败状态
unknown|无法获取pod状态，网络问题等

重启策略restartpolicy：

Always：自动重启
onfailure：容器终止且退出码不为0，由kubectl自动重启
never：不重启

重启策略在不同的控制方式下设置不一样：

RC/daemonset：always，必须保证持续运行
job：onfailure/never，因为是全部执行完毕后就结束，所以不重启

#### 健康检查和服务可用性检查

主要是用两个探针来实现。

livenessProbe：判断容器是否为running状态，是否存活。默认为success，只有设置了才会去探查，根据重启策略重启。

readnessProbe：判断容器服务是否可用，为ready状态？只有ready才可以对外提供服务，否则的话从service后端的endpoint列表里格里拉出去，避免用户进入服务不可用的pod实例。

probe实现方法：

execAction：执行命令，返回0则正常。执行cat \/tmp/health等。

TCPSocketAction：通过容器的IP:PORT来判断加你差，设置一个port

HTTPGetAction：通过get某个地址的某个多端口返回的状态码判断是否健康（200~400都是健康）。

默认参数：

initialDelaySeconds：什么时候开始首次检查？

timeoutSeconds：等待多久后判定为无法提供服务？

#### Pod调度(运行节点选择控制)

RC(选一个标签) ---> RS(选多个标签) ---> deployment(更自动的部署)

deployment实际上是通过RS来实现副本自动控制的，所以用deployment。

调度时可能需要实现以下内容：

1. 想要放在特定的节点上

通过制定NodeSelector来实现。

2. 想要放在特定节点，但是目标不符合条件，资源不足；如果要选择多种合适的目标节点？

使用NodeAffinity节点亲和性设置。

3. 不同节点之间，一些容器就要放一起，一些不能一起，怎么办？

使用PodAffinity实现

4. 有状态集群怎么调度？虽然节点看起来都一样，但是他挂载的数据是有状态的，需要顺序执行。

使用statefulSet来特殊控制副本，Pod绑定具体的PV和PVC做持久化存储。

5. 需要调度仅创建一个Pod副本，如系统的监控log，只需要一个？

用daemonset实现。

6. 批处理作业，创建多个Pod来协作，都完成后整个作业结束。也就是一批只做一次的任务的调度怎么做？

使用Job来做，并延伸出了CronJob。

##### deployment

自动部署+维持副本数量+监控副本数量
kind:Deployment
spec.replicas指定数量
spec.template指定pod模板，如.metadata,.spec.container等

##### NodeSelector定向调度

先对Node打上标签
 kubectl label nodes <node-name> <label-key>=<label-value>
 
然后在Pod定义中执行nodeselector
.spec.template.spec.nodeSelector: zone=north

会对标签对应的内容挑选一个可用Node进行调度。

##### NodeAffinity:节点亲和性调度

亲和性调度这种调度方式更具表达力。

使用软限制、优先采用等方式，代替硬限制。

节点亲和性**可以依据节点上其他Pod的标签来限制**，而不是非要用当前Pod来做限制，所以可以**描述Pod之间的亲和或互斥关系**。

NodeAffinity是用来替换NodeSelector的，有两种表达方式：

1. required during scheduling ingnored during excecution

必须满足规则才可以调度到节点上，相当于硬限制。

2. preferred during scheduling ingnored during excecution

优先调度到满足当前规则的节点，软限制。

两种方式中的ingnored during excecution表示的是：

如果Pod所在的Node在Pod运行期间(excecution)标签发生了变更，不再符合亲和性需求，则可以忽略label变化，继续运行Pod在该节点上。

设置在.spec.affinity.nodeAffinity.requiredduringschedulingingnoredduringexcecution.nodeselectorterms中做一系列的matchexpressions字段匹配

而不是在template中配置，和template是同级的。

匹配规则：

nodeSelector+nodeAffinity，则两个都要满足

nodeAffinity指定多个nodeSelectorTerms，满足一个Terms就好

nodeSelectorTerms有多个matchExpressions，一个节点必须满足所有expression

##### PodAffinity

根据节点正在运行的Pod的标签而不是Node标签来做调度，要求对Node和Pod都进行匹配。

**可以描述为：如果在具有标签X(区域标签，通过topologyKey指定)的Node上运行一个或多个符合条件Y的Pod，那么Pod应该运行在这个Node上。**

设置参数和Node一样，也是2个：required during scheduling ingnored during excecution和preferred during scheduling ingnored during excecution

亲和性案例

```
// 制定一个security=SI的亲和标签，同时topologyKey=kubernetes.io/hostname，即Node要有这个topologyKey标签
apiVersion: v1
kind: Pod
metadata:
	name:affinity
spec:
	affinity:
		podAffinity:
			requiredduringschedulingingnoredduringexcecution:
			- labelSelector:
				matchExpressions:
				- key: security
					operator: In
					values:
					- S1
			topologyKey: kubernetes.io/hostname
```

互斥性案例(基本一样，不过是把podAffinity修改一下，改成podAntiAffinity)

```
// 不能放在节点区域标签为hostname的节点上
apiVersion: v1
kind: Pod
metadata:
	name:affinity
spec:
	affinity:
		podAntiAffinity:
			requiredduringschedulingingnoredduringexcecution:
			- labelSelector:
				matchExpressions:
				- key: security
					operator: In
					values:
					- S1
			topologyKey: kubernetes.io/hostname
```

##### 污点+容忍

Affinity是为了实现Pod调度到某些更好的节点上，Taint相反，他让Node拒绝Pod运行。

Taint和Toleration一起使用，让Pod避开不合适的Node。

怎么配合使用？

给Node加一个Taint属性，如果Pod上声明的Toleration可以容忍这个Taint，那么才可以在这个Node上运行。

```
// noSchedule表示的是效果，不进行调度，还有noExecute不执行
kubectl taint nodes node1 key=value:NoSchedule
```

```
// Pod上，指的是能容忍key=value:noSchedule的Node。
tolerations:
- key: "key"
	operator: "Equal"
	value: "Equal"
	effect: "NoSchedule"
```

##### DaemonSet：在每个Node上都调度一个Pod

实例：日志采集程序fluentd，logstach；性能监控程序prometheus node exporter等。

也可以对DaemonSet进行滚动升级，在yaml中添加参数：
```
apiVersion
kind: DaemonSet
metadata
spec:
	updateStrategy:
		type: RollingUpdate
```

##### Job批处理

几种模式：

- Job 模块 扩展 模式：

一个Job对象对应一个待处理的工作，Job和工作一一对应。适合工作数少，工作要处理的内容多的场景。

- Queue with Pod Per Work Item

采用任务队列存放工作，一个Job去完成这些工作，Job怎么完成呢？通过**启动N个Pod去实现N个工作**。工作->任务队列->Job单个->分出多个Pod

- Queue with variable Pod Count

采用任务队列存放工作，一个Job去完成这些工作，Job怎么完成呢？通过**启动M个Pod去实现N个工作**，Pod数量不一定就和工作数量一样。工作->任务队列->Job单个->分出多个Pod

Job可以分为三类：

- 没并行的Job

一个Job启动一个Pod

- 并行的Job，使用完成数计数

Job启动多个Pod。

设定.spec.completion>0，当正常结束的Pod数量达到参数时，Job结束。

设定.spec.parallelism控制并行度，即同时启动多少个Pod来处理工作。

- 并行Job，使用任务队列

额外设置一个任务队列，存放工作

###### CronJob 定时任务

```
apiVersion:
kind: CronJob
metadata:
spec:
	schedule: "*/1 * * * *" //表示每隔1分钟触发任务， a/b的含义是第一次触发在a，隔b再触发。
	jobTemplate:
		spec:
```

至此，所有的Pod调度器都解释清楚了。

#### 初始化容器 init container

初始化容器这个容器的存在是为了什么？

是为了在启动应用容器之前启动多个初始化容器，把应用的预置条件配置好。**初始化容器不一样的地方在于他只运行一次**。

初始化容器和应用容器的不同：
1. 执行顺序，初始化早于应用容器，所有初始化容器的执行也有顺序
2. 都有资源限制配置，但是初始化容器会选所有初始化容器中的配额最大值
3. 初始化容器不能设置readinessProbe探针，因为成功运行后才能运行普通容器

Pod重启后，初始化容器会重启，Pod重启在这几个场景：
1. 初始化容器的image更新，初始化容器会重启，从而Pod重启。而普通容器的image更新不会重启Pod，只会重启应用容器。
2. Pod的infrastructure容器(也就是pause容器)更新，Pod会重启
3. Pod中所有应用容器终止，并且重启策略=always，Pod重启

#### 重要!!!Pod升级和回滚(版本控制)

更新服务时需要停止所有服务相关Pod，如果集群过大，这个过程会很复杂，时间长+可能出错。会导致长期的服务不可用，所以才引出了滚动升级功能。

如果是Deployment创建的Pod，可以通过修改dep.yaml中的内容，然后apply，那么就会进行自动更新，如果错误，可以回滚到旧版本。

可以通过
```
kubectl rollout status deployment/depname
```
查看对应内容的deployment的更新过程
```
kubectl describe deployment/depname
```
查看详细事件信息

整个deployment的更新流程？

创建一个RS->根据需求创建副本->收到更新命令->创建新的RS，数量=1->就的RS缩到2->new RS=2,old RS=1->new RS=3,old RS=0->完成

上面说的是直接修改deployment来应用，更新策略其实可以在yaml中的spec.strategy设置

- Recreate

spec.strategy.type = Recreate，表示更新时，先杀掉所有Pod，然后创建新的

- RollingUpdate

spec.strategy.type = RollingUpdate，滚动更新。

spec.strategy.RollingUpdate.maxUnavailable: 指定更新时不可用状态Pod的数量上限

spec.strategy.RollingUpdate.maxSurge:指定Pod总数可超过期望副本数的最大值

**deployment的回滚**

```
// 查看deployment历史
kubectl rollout history deployment/depname
// 得到历史列表
REVISION   CHANGE-CAUSE
1			操作代码1
2			操作代码2
3			操作代码3
// 回滚
kubectl rollout undo deployment/depname --to-revision=2
```

DaemonSet的更新策略：

- OnDelete

默认升级策略。创建好新的配置后，只有用户手动删除旧版本Pod，才触发新建操作。

- RollingUpdate

旧版本自动被杀掉，然后创建新的。

但是两个不同：1.不支持查看DS的更新记录2.回滚不能直接通过kubectl rollback指令执行(因为看不了更新记录)，只能重新提交旧版本apply

StatefulSet的更新策略：

也逐渐支持OnDelete、RollingUpdate、partition策略，能保留更新历史，可直接回滚历史版本。

#### Pod扩缩容(副本控制)

手动修改deployment的spec.relicas指标

或者使用自动扩容机制(注意HPA的探测周期可调)

早期的自动扩缩容数据来源于Heapster，heapster可以理解为一个数据收集器，他收集Node上的cadvisor数据。

HPA工作原理：

Metrics Server持续采集所有Pod副本的指标数据->HPA持续获取MS的API上提供的数据信息->HPA根据用户定义的扩缩容规则计算目标副本数->数目不同时进行扩缩容。

- 指标类型

指标由Master的kube-controller-manager服务持续监测，包含Pod资源利用率+Pod自定义指标+Object自定义指标或外部指标

- 扩缩容算法

目标副本数 = 当前副本数 × （当前指标值 / 期望指标值），往上取整。

当目标与1十分接近，可以设置一个容忍度0.1，在结果的-0.1~+0.1之间都不进行扩缩容。

HPA排除了部分Pod的指标，不计入运算：

正在删除的/指标无法获得的/刚启动没到特定时间的/还没ready的

此外还加入了平滑，系统会记录一段时间里最好的副本选择，从而避免短时抖动。

- yaml指定
```
apiVersion:autoscaling/v1
kind:HorizontalPodAutoscaler
matadata:
spec:
	scaleTargetRef:
		apiVersion:
		kind:deployment
		name
	minReplicas:
	maxReplicas:
	targetCPUUtilizationPercentage:
```
```
apiVersion:autoscaling/v2beta2
kind:HorizontalPodAutoscaler
matadata:
spec:
	scaleTargetRef:
		apiVersion:
		kind:deployment
		name
	minReplicas:
	maxReplicas:
	metrics:
	- type: Resource
		resource:
			name: cpu
			target:
				type: Utilization
				averageUtilization: 50
	- type: Pods
		pods:
			metric:
				name:packets-per-second
			targetAverageValue: 1k
```

- 自定义HPA

预先部署自定义的Metrics server，可以用prometheus、Azure的Adapter自定义实现。

架构图实际上就是答辩ppt里的多色图：Prometheus->custom metrics server->metrics aggregator->API server->HPA->Deployment->Pod调整

### Service

为一组有相同功能的容器应用提供统一的入口地址，把请求分发到后端的应用上。

主要是通过selector把相应的deployment等内容选中组合在一个入口。

```
//必须存在的字段
apiVersion:
kind:Service
metadata:
	name:
	namespace:
spec:
	selector:[]
	type:
	cluster IP:

```

一开始是直接通过containerPort设置提供服务的容器端口，但是存在问题。

在deployment的spec.template.spec.container.ports.- containerPort直接设定

这样可以通过kubectl get pod -l app=webapp -o yaml | grep podIP查看Pod的IP地址和端口号。

问题：1.Pod所在的IP地址是不可靠的，如果Pod所在的Node发生故障，Pod的IP地址会发生变化。2.如果容器本身是分布式的，有多个实例，那么服务访问就有问题，多个端口，还需要前端设置一个负载均衡器做请求分发。

这个负载均衡器实际上就是Service。

Service的IP的和端口确定后，内部访问这个Service就直接通过cluster IP:port来进行。对这个端口的访问就直接转发到后端的两个容器上。

```
apiVersion:
kind:Service
metadata:
spec:
	ports:
	- port: 8081//访问service时的外部端口号
	  targetPort:8080//绑定的容器的暴露的服务端口号
	selector:
	  app: webapp//指定后端的容器
```

所以service的yaml主要做的就是绑定selector对应的容器的对应端口号，然后把这个端口号绑定在自己的外部端口号上。

有了Service这个负载均衡器，然后怎么进行负载分发呢？

1. 轮询模式(默认)： 把请求转发到各个绑定的Pod

2. 基于客户端IP的会话保持模式：第一次时把某客户端发起的请求转发到后端某个Pod上，之后从相同客户端发起的都转发到这个Pod上。所以取决于第一次的绑定。可以通过sessionAffinity来绑定亲和哪个Pod的clusterIP

-  Service多端口

```
spec:
	ports:
	- name:
	  port:
	  protocol:
	- name:
	  port:
	  protocol:
```

#### 外部访问Pod或Service

在Pod的yaml中指定spec.template.containers.ports.hostPort

不常用，多副本时会冲突。

#### Service端口映射到物理机

设置Service的spec.ports.nodePort，默认在30000~32767之间选一个。

对应的防火墙上也要设置到权限，使得外部能访问。才能用物理机IP:nodePort进行访问。

### DNS服务

集群内是通过ClusterIP进行访问的，这个对应的映射关系由集群内的DNS服务提供。目前采用的是CoreDNS。

### Ingress(重要,服务发现和负载均衡)

Service暴露端口有个问题，如果Service很多，那么要开放很多端口，怎么管理？

引入Ingress，定义一个Ingress策略+Ingress Controller(相当于边缘路由器)就好了。

1. 部署ingress Controller

实现逻辑：监听API Server，获取全部Ingress策略的定义->基于定义，生成nginx配置文件->加载配置文件，建立关联关系

2. 定义ingress策略

```
// 这里定义一个从website.com/demo的域名到webapp的service的8080/demo端口的映射
apiVersion:
kind:Ingress
matadata:
spec:
	rules:
	- host: website.com
	http:
		paths:
		- path: /demo
		  backend:
		    serviceName: webapp
			servicePort: 8080
```


# Kubernetes核心组件(重要)

## 最重要的API Server

提供K8S各类**资源对象的增删改查+Watch+K8S Proxy API的HTTP Rest接口**，是集群内**各模块间数据交互和通信的中心枢纽**。

还提供**集群管理**API/**资源配额控制**API/**集群安全**机制。

在8080端口提供REST服务，6443端口提供HTTPS安全端口加强接口访问安全性。

curl localhost:8080/api查看版本信息。此后的一系列地址可以查看更多细节。

kubectl proxy命令可以启动内部代理 设置访问白名单 设置服务不可访问--reject-paths

API Server需要很高性能，因为是集群核心，主要运用这些方式：

- 高性能底层代码，协程+队列来做并发，超强多核处理能力，快速并发处理。

- List接口结合异步Watch接口，解决同步问题，提升响应灵敏度。

- 高性能etcd数据库，而不是传统关系型数据库。

### 最重要的List-Watch

Etcd存储集群的数据信息，apiserver作为统一入口，任何对数据的操作都必须经过apiserver。

客户端(kubelet/scheduler/controller-manager)通过list-watch监听apiserver中资源(pod/rs/rc等等)的create,update和delete事件，并针对事件类型调用相应的事件处理函数。

各模块间需要和API Server轮询资源对象状态，API Server需要向各模块发出通知，这两块怎么实现，通过List-Watch。

针对这个场景，会发现几个问题：

轮询，怎么保证apiserver压力不会太大，保证实时性？

通知各模块，怎么保证消息可靠，解决端口占用问题？

List-Watch = List + Watch

list就是调用资源的list API罗列资源，基于**HTTP短链接**实现；watch则是调用资源的watch API监听资源变更事件，基于**HTTP 长链接**实现

K8S的informer模块封装list-watch API，用户需要指定资源，编写事件处理函数AddFunc,UpdateFunc和DeleteFunc等。

informer首先通过list API罗列资源，然后调用watch API监听资源的变更事件，并将结果放入到一个FIFO 队列。

队列的另一头有协程从中取出事件，并调用对应的注册函数处理事件。

Informer还维护了一个只读的Map Store缓存，主要为了提升查询的效率，降低apiserver的负载。

<div align="center"> <img src="https://pic4.zhimg.com/80/v2-5263be53b45fdbea59e9812d9df627b7_720w.jpg" width=""> </div><br>

```
c.NodeLister.Store, c.nodePopulator = framework.NewInformer(
    c.createNodeLW(),                                            ...(1) LW操作函数
    &api.Node{},                                                 ...(2) 
    0,                                                           ...(3)
    framework.ResourceEventHandlerFuncs{                         ...(4) 事件处理函数触发入口
        AddFunc:    c.addNodeToCache,                            ...(5)
        UpdateFunc: c.updateNodeInCache,
        DeleteFunc: c.deleteNodeFromCache,
    },
)  
```

- 消息可靠性

避免因消息丢失而造成状态不一致场景。

list API可以查询当前的资源及其对应的状态(即期望的状态)，客户端通过拿期望的状态和实际的状态进行对比，纠正状态不一致的资源。

Watch API和apiserver保持一个长链接，接收资源的状态变更事件并做相应处理。

若某个时间点连接中断，就有可能导致消息丢失，所以需要通过list API解决消息丢失问题。

我们可以认为list API获取全量数据，watch API获取增量数据。虽然仅仅通过轮询list API，也能达到同步资源状态的效果，存在上面提到轮询的缺点：开销大，实时性不足的问题。

list实际上是用于辅助修正的，避免watch出问题了。

- 消息实时性

每当apiserver的资源产生状态变更事件，都会将事件及时的推送给客户端，从而保证消息的实时性。

- 顺序性

K8S在每个资源事件中都带一个resourceVersion的标签，这个标签是递增的数字。

所以当客户端并发处理同一个资源事件时，它就可以对比resourceVersion来保证最终的状态和最新的事件所期望的状态保持一致。

- 高性能

List-watch通过异步通知达到高性能的特点，watch作为异步消息通知机制，复用一条长链接，保证实时性的同时也保证了性能。

### API Server架构

API层->访问控制层->注册表层->etcd数据库

- API层

提供一系列REST接口，如资源对象CRUD和Watch这五类主要API，还有/healthz、/logs、/ui(dashboard)、/metrics等运维监控相关接口。

- 访问控制层

对用户验明身份，鉴权，核准用户的资源对象访问权限（这些是针对用户的）；准入控制（针对资源，根据配置的资源访问逻辑判断能否允许访问）。

- 注册表层

保存所有资源对象在注册表中。包括对象的类型、创建方法、版本转换方式、如何把资源编解码为JSON或ProtoBuf格式存储。

- etcd数据库

KV数据库，持久化存储资源对象。**etcd Watch API 十分重要**。，List-Watch是高性能的资源对象实时同步机制。

- 其他设计细节：

版本迭代和版本数据转换。处理的是同一对象不同版本的数据转换问题以及API接口版本兼容问题。

使用版本号控制接口兼容区分，数据转换使用internal版本，各个版本对象都能转换成internal版本对象，做各个版本对象的数据转换中介，从两两连接的混乱拓扑变成星状图。

### API Server提供的另一个独特接口——K8S Proxy API接口

是作为代理，把收到的REST请求转发到Node上的kubelet的ERST接口，由kubelet响应。

```
//获取节点上所有Pods, 获取的信息是来自Node的,可能与etcd的有时间偏差
curl localhost:8080/api/v1/proxy/node/node1/pods
//访问pod
curl localhost:8080/api/v1/proxy/node/node1/pods/{name}
//访问接口
curl localhost:8080/api/v1/proxy/node/node1/pods/{path:}
```

这类接口主要是用来在外部冯文某个Pod容器的HTTP服务，用Proxy API实现，比如用来逐一排查Service的Pod哪个异常了。

同时service也可以使用

```
curl localhost:8080/api/v1/provy/namespace/{namespace}/services/{name}
```

### 集群模块间通信

- kubelet与API server

kubelet定期调用REST接口报告自身状态，API server把节点信息更新到etcd。

kubelet通过Watch接口监听Pod信息，如果有心副本调度到本节点，则执行Pod创建启动逻辑;若删除,则删除,若修改,则修改。

- kube-controller-manager与API server

Node controller通过Watch接口实时监控Node信息，并做处理。

- kube-scheduler与API server

通过Watch接口监听，如果有新建Pod副本的信息，会检索符合的Node列表，执行Pod调度，绑定Pod到Node上。

- 优化

各模块都采用本地缓存，缓存数据，各模块定时使用List-Watch方法获得想要的信息，缓存在本地缓存。

某些情况下通过访问缓存来直接操作，而不用直接访问API Server。从而降低API Server的访问压力。

## Controller manager

各个controller通过LW接口实时监控特定资源变化，并应对故障，保持状态为期望状态。共有8类。

### Replication Controller

副本控制器，不同于资源对象RC，确保任何时候与RC关联的Pod副本数保持在预设值。但注意RC的重启策略必须是always。

因为有了副本控制器，所以最好不要直接创建Pod，而使用deploymengt等来构建。

综上：

副本控制器做的是：

1. 保持集群中有且只有N个Pod实例

2. 调整RC的replicas属性来实现扩缩容

3. 通过改变Pod模板来实现滚动升级

场景：

1. 重新调度，即Node故障或Pod副本异常终止，能提供RC副本数保证

2. 弹性伸缩，手动的扩缩容修改

3. 滚动更新，副本控制器的流程是逐个替换Pod的方式，通过依次增新Pod，减旧Pod来实现所有副本的更新。直到旧RC副本为0，然后删除。

### Node Controller

kubelet启动时会在API Server注册Node信息，并定时汇报状态，API Server更新信息到etcd中，包括健康状态，资源，名称，地址信息，OS版本，docker版本，kubelet版本。

Node Controller通过API Server获取Node信息，管理和监控集群中各个Node。

工作流程：

1. 如果设置了--cluster-cidr，回味Node生成CIDR地址，防止地址冲突。

2. 逐个读取Node信息，和本地保存的nodeStatusMap的节点信息比较。

2.1. 如果没有收到信息或第一次收到或过程中节点变得非健康：则保存信息，记录当前探测时间和节点变化时间

2.2. 收到新节点信息，保存新状态，记录当前探测时间和节点变化时间

2.3. 收到节点信息，但节点状态没变化，保存上次节点的变化时间

3. 如果没有收到节点信息，则设置节点状态为Unknown

4. 逐个读取节点信息，未就绪节点入队列，准备删除；并同步正常节点更新的信息。

### ResourceQuota Controller 资源配额管理器

确保指定资源对象任何时候都不会超量占用系统物理资源，避免某些设计缺陷导致系统紊乱/宕机。

层次为：容器级别、Pod级别、namespace（多租户）级别(pod数量,RC数量,Service数量,ResourceQuota数量,Secret数量,PV数量)

资源配额通过API Server的准入控制来控制（ResourceQuota(作用于namespace)和LimitRanger(作用于Pod和容器)两种方式）。

每次请求创建资源时，准入控制会计算当前配额使用情况，从etcd里读，不符合那就创建失败。

符合那就更新目前配置的资源量到etcd。而这些信息从ResourceQuota Controller中更新得到，最后存入etcd。

### Namespace Controller

定时读取namespace信息，如果有namespace要优雅删除，则标记，更新到etcd。准入控制里同时会组织改namespace继续创建新资源。

### Service controller 和 endpoint controller

service/endpoint/pod关系：endpoint为service对应的所有Pod副本的访问地址(具体为PODIP:PORTS)

endpoints对象在Node的kube-proxy上使用，proxy获取每个endpoints，实现Service负载均衡。

endpoint controller负责生成和维护所有endpoints对象信息。监听service和Pod副本变化，变化则同步更新service，Pod对应的endpoints对象。

Service controller监听service，如果是loadbalance类型，那么确保在外部云平台上对应的loadbalance实例更新正确，并更新路由转发表（根据endpoint）

### ServiceCount controller 和 Token controller

ServiceCount controller保证每个namespace至少有一个default service account。

如果启动了APIserver私钥，Token controller会被创建，也监听serviceaccount。确保serviceaccount至少有一个secret对象。

如果没有secret对象，那么就会用私钥创建一个Token，用token+CA证书+namespace名称产生secret，放入对应serviceaccount中。

此外token controller监听secret对象和处理。

## Scheduler原理

承上：负责接收Controller manager创建的新Pod，为他确定目标Node

启下：安置ok后，Node上kubelet接管后续工作。

具体的，Scheduler通过调度算法和策略把待调度Pod列表中的Pod从可用Node列表里选择合适Node并调度过去，同时更新到etcd。

设计待调度Pod列表，可用Node列表，调度算法和策略三大内容。

!!!调度流程两步：预选调度(列出合适候选,预选策略)+确定最优节点(优选策略,高分者胜出)，实际上用了预选策略+优选策略。

预选策略：NoDiskConflict(gcePersistentDisk和AWSElasticBlovkStore两类挂在卷类型,看看有没有磁盘冲突)/PodFitsResource(是否满足Pod需求,如内存CPU资源)/PodSelectorMatches(Pod的nodeSelector标签选择器要求是否满足,如node必须要被打伤某个k-v标签值才能作为目标)/PodFitsHost(nodeName域指定的节点名称是否一致)/CheckNodeLablePresence/CheckServiceAffinity/PodFitsPorts(端口列表是否占用)

优选策略:LeastRequestPriority(选出资源CPU和内存消耗最小的节点,计算使用率的剩余量之和)/CalculateNodeLabelPriority(对配置的标签的匹配度)/BalencedResourceAllocation(选出CPU和内存使用率最均衡的,计算二者差值,要使用率尽可能近,不能不平衡)

## kubelet原理

每个Node都有kubelet进程，用于处理下发到本节点的任务，管理Pod和容器。每个kubelet都在API Server注册节点信息，定期汇报，使用cAdvisor监控容器和节点资源。

- 节点管理

自注册节点信息到API server，定时更新状态给API server

-  Pod管理

监听etcd-API server的Pod信息变化。

如果要删除Pod:

那就删除，并通过Dokcer Client删除Pod中的容器。

如果是创建或修改Pod任务：

为Pod创建数据目录->从API Server读取Pod清单->为该Pod挂载卷->下载Pod用到的secret->检查已经运行在节点上的pod,如果没有容器或者Pause容器没有启动(即未就绪),则停止Pod内所有容器进程.->每个Pod创建pause容器,Pause容器接管所有业务容器的网络.->为Pod中的每个容器做这些(为容器计算hash值,查看名称对应的容器hash,不一样则停止容器和与之关联的Pause容器,如果一样则不作处理/如果容器不终止并且没有指定的重启策略则不处理/否则调用Docker Client下载镜像,运行容器)

-  容器健康检查

LivenessProbe(是否健康,不健康就删除容器,没探针的话默认都健康)和ReadinessProbe(判断是否启动完成,可以接受请求,如果失败则由Endpoint Controller把这个容器所在Pod的endpoint条目删除)

执行方式：执行命令(cat /tmp/health)/TCP检查端口/HTTP检查端口

- cAdvisor监控资源

4194端口暴露一个简单UI。弃用，想用可以用daemonSet新建一个cadvisor。现在监控用metrics server和custom metrics。

## kube-proxy

Service是对一组Pod的抽象，创建Service时会创建虚拟IP，客户端访问虚拟IP，Service把请求转发到后端Pod。Service的作用实际上就是kube-proxy完成的。

proxy是Service的代理+负载均衡器。核心功能就是把某个到Service的访问转发到后端多个Pod实例上。Service的Cluster IP和NodePort等就是proxy通过iptables的NAT转换实现的。

proxy目前的默认模式是iptables模式，核心功能变成：watch apiserver接口追踪service和endpoint变化，更新iptables信息，客户的请求流量通过iptables的NAT网络地址转换(IP端口名称的映射关系)直接路由到目标Pod。

iptables模式完全在内核态，不用专户单到用户态的proxy，所以性能更强。

缺陷：service，pod大量增加，规则数大，性能下降，所以引入了IPVS，即IP虚拟服务器，提供高性能负载均衡，可以无限规模扩张，数据结构用哈希表。

### IPVS

## 集群安全

包括认证授权，准入控制，secret机制。

逻辑是认证鉴权-准入控制-得到响应。

## API server认证管理

如何认证识别客户端身份？访问权限怎么授予？即认证+授权问题

提供三种级别认证方式：

1. HTTPS CA(可信第三方)根证书认证,和HTTPS的SSL认证差不多

2. HTTP token认证.Token存放信息，在API调用时,http头里加入token就好

3. HTTP base认证,用户名+冒号+密码用BASE64算法编码后放在头认证域，发给服务端。

## API Server授权管理

给不同用户不同的访问权限:

- AlwaysDeny

- AlwaysAllow(默认)

- ABAC基于属性的访问控制

指定授权策略文件，文件中每一行以map形式提供访问策略对象，可以设置apiVersion,kind:Policy,spec属性。

ABAC中，收到请求，识别出携带的策略对象属性，然后匹配策略文件，至少一条匹配成功就通过授权。

Serviceaccount会自动生成ABAC用户名，创建新的namespace时会产生serviceaccount，携带ABAC用户名，把这个写入授权策略就好。

- RBAC基于角色的访问控制

引入Role、ClusterRole、RoleBinding、ClusterRoleBinding四个资源对象，以yaml形式编写构建。

Role绑定namespace，在namespace中用role定义角色权限

ClusterRole绑定集群内角色,可以访问集群范围内的资源、全部命名空间的资源等

RoleBinding、ClusterRoleBinding绑定有权利的角色和目标对象，使目标(User，Group，Service Acount)得到权力。

逻辑是：(User，Group，Service Acount)->通过RoleBinding->再到Role->再到对Pod的操作get watch list等

- Webhook调用外部REST服务对用户授权

实现HTTP接口，k8s会调用这个接口去外部REST服务中对用户授权。此时API Server变成客户端，远端授权服务网站变成授权服务端。

- Node专用于kubelet请求访问控制

## 准入控制

准入控制链检查，通过检查才能得到相应。

## Service Account

一种账号，给Pod里的进程用，属于当前namespace。这些进程要去访问API Server时，就用serviceaccount来做身份认证。Service Account包含多个secret对象，是用于不同访问目标的身份鉴权协助。

## Secret 私密凭据

保管私密数据，密码、token、keys等。放在secret对象里更安全，便于使用和分发。

怎么用？创建pod时指定serviceaccount从而自动使用secret/挂载secret到pod(volume)/指定pod的imagepullsecret额外引用这个secret

## K8S 网络原理

每个Pod有一个独立的ip地址，所有pod都在联通的网路空间里，pod内容器共享一个网络堆栈(pause容器+业务容器，从而pod内容器可以用localhost访问对方端口)。

K8S网络和Dokcer动态端口映射方式的区别？

Docker的更复杂，端口管理复杂，访问者看到的IP和实际IP不同，应用配置很复杂。

为了支持这个简单K8S模型，要求所有容器/节点都可以不用NAT时和别的容器/节点通信，容器的地址和别人看到的地址是同一个。

## Docker网络基础

### 网络命名空间

通过网络命名空间把独立的协议栈隔离到不同namespace中，相互独立、

各自namespace可以有自己的独立路由表，iptables。

对于独立的各个网络协议栈，只能通过veth设备对(veth pair)来做通信，连接两个网络协议栈。veth pair就是为了在不同网络namespace中通信创建的，成对出现。

网络连起来了，那么怎么把主机间通信连起来？用网桥，一个二层虚拟网络设备，使得网络接口间报文能互相转发。通过mac地址识别转发。

可以通信了，那怎么对特定数据包进行处理？引入Netfilter和iptables。

### Docker网络模式

4类：

- host(--net=host)

- container(--net=container:NAME_or_ID)

- none(--net=none)

- bridge(--net=bridge)

k8s下只使用这个。

dockerdaemon第一次启动，创建虚拟网桥docker0，给他分配一个子网，对于docker创建的每个容器，都会创建一个虚拟的veth设备对，一段在网桥，一端用网络namespace映射到容器内的eth0设备，给eth0分配一个IP地址。

## K8S网络实现

这个比docker复杂多了

需要解决容器间/Pod间/pod和service间/集群外和集群内之间的通信。

