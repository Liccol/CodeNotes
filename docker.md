# 容器 与 Docker

容器 = cgroup + namespace + rootfs + 容器引擎(docker 中就是 docker engine)

## Docker和虚拟机的区别

<div align="center"> <img src="https://upload-images.jianshu.io/upload_images/12979420-a562cd670f2b8b02?imageMogr2/auto-orient/strip|imageView2/2/w/529/format/webp
" width=""> </div><br>

- (docker抽象层更少,效率更高,更高资源利用率)虚拟机的Guest OS层，还有Hypervisor层在Docker上已经被Docker Engine层所取代.

Guest OS 是虚拟机安装的操作系统，是一个完整的系统内核;Hypervisor可以理解为硬件虚拟化平台，它在Host OS后以内核驱动的形式存在。

Docker容器上的程序，直接使用的都是实际物理机的硬件资源。因此在cpu、内存、利用率上，Docker将会在效率上具有更大的优势。

- (资源隔离方式不同,带来不同的性能开销)虚拟机实现资源的隔离的方式是利用独立的Guest OS+Hypervisor虚拟化实现的；Docker简单很多，利用的是目前当前Linux内核本身支持的容器方式。

- (启动速度不同)Docker秒级启动。Docker直接利用系统内核，避免了虚拟机启动时所需要的系统引导时间和操作系统运行的资源消耗。

- (兼容性)Docker能够高效地部署和扩容，Docker容器几乎可以在任意平台上运行，包括虚拟机、物理机、公有云、私有云、个人电脑、服务器等。

## namespace(资源隔离)

Linux Namespace 是kernel 的一个功能，它可以**隔离**一系列系统的资源，比如PID(Process ID)，User ID, Network等等。

linux内核实现namespace的主要目的，就是为了实现轻量级虚拟化技术服务。

在同一个namespace下的进程感知彼此的变化，而对外界的进程一无所知。

这样就可以让容器中的进程产生错觉，仿佛自己置身一个独立的系统环境中，以达到隔离的目的。

linux内核提拱了6种namespace隔离的系统调用：

namespace|系统调用参数|隔离内容
UTS|CLONE_NEWUTS|主机名或域名
IPC|CLONE_NEWIPC|信号量、消息队列和共享内存
PID|CLONE_NEWPID|进程编号
Network|CLONE_NEWNET|网络设备、网络战、端口等
Mount|CLONE_NEWNS|挂载点（文件系统）
User|CLONE_NEWUSER|用户组和用户组

Namesapce 的API主要使用三个系统调用：

- clone() - 创建新进程。根据调用参数来判断哪种namespace被创建，而且它们的子进程也会被包含到namespace中。

使用这些API时需要指定一下6个参数中的一个或多个，通过 | 操作实现。

这6个参数分别是CLONE_NEWUTS、CLONE_NEWIPC、CLONE_NEWPID、CLONE_NEWNET、CLONE_NEWNS、CLONE_NEWUSER。

- setns() - 将进程加入到namespace中

在进程都结束的情况下，也可以通过挂载的形式把namespace保留下来，保留namespace的目的是为以后有进程加入作准备。

在docker中，使用docker exec命令在意境运行的容器中执行新的命令，就需要用到该方法。

通过setns()系统调用，进程从原先的namespace加入到某个已经存在的namespace。

为了使新的pid namespace生效，会在函数执行后使用clone()创建子进程继续执行新命令，让原先的进程结束运行。

- unshare() - 将进程移出某个namespace

调用unshare()的主要作用就是，不启动新进程就可以起到隔离的作用，相当于跳出原先的namespace进行操作

这样，就可以在原进程进行了一些需要隔离的操作。

linux中自带unshare命令，就是通过unshare()系统调用实现的，**docker目前并没有使用这个系统调用**。

### UTS namespace

UNIX Time-sharing System，提供了主机名与域名的隔离，这样每个容器就可以拥有独立的主机名和域名了。为什么要隔离主机名？因为主机名可以代替IP来访问。如果不隔离，同名访问会出冲突。

在网络上可以被视为一个独立的节点，而非宿主机上的一个进程。

docker中，每个镜像基本都以自身所提供的服务名称来命名镜像的hostname，且不会对宿主机产生任何影响。

### IPC namespace

Inter-Process Communication。进程间通信IPC资源包括常见的信号量、消息队列和共享内存。

申请IPC资源就申请了一个全局唯一的32位ID，所以IPC namespace中实际上包含了系统IPC标识符以及实现POSIX消息队列的文件系统。

在同一个IPC namespace下的进程彼此可见，不同IPC namespace下的进程则互相不可见。

目前使用IPC namespace机制的系统不多，其中比较有名的有PostgreSQL。Docker当前也使用IPC namespace实现了容器与宿主机、容器与容器之间的IPC隔离。

### PID namespace

PID namespace隔离非常实用，它对进程PID重新标号，即两个不同namespace下的进程可以有相同的PID。每个PID namespace都有自己的计数程序。

内核为所有的PID namespace维护了一个树状结构，最顶层的是系统初始时创建的，被称为root namespace，它创建的新PID namespace被称为child namespace(树的子节点)，构建节点父子关系。

通过这种方式，不同的PID namespace会形成一个层级体系。所属的父节点可以看到子节点中的进程，并可以通过信号等方式对子节点中的进程产生影响。反过来，子节点却不能看到父节点PID namespace中的任何内容。

由此产生如下结论：

- 每个PID namespace中的第一个进程“PID 1”，都会像全通Linux中的init进程一样拥有特权，其特殊作用。

- 一个namespace中的进程，不可能通过kill或ptrace PID影响父节点或者兄弟节点中的进程，因为其他几点的PID在这个namespace没有任何意义。

- 如果你在新的PID namespace中重新挂载/proc文件系统，会发现其下只显示同属一个PID namespace中的其他进程。

- 在root namespace中看到所有的进程，并且递归包含所有子节点中的进程。

到这里，可能已经联想到了一种在Docker外部监控运行程序的方法了，就是监控Docker daemon所在的PID namespace下的所有进程及子进程，再进行筛选即可找到想监控的程序。

### network namespace

主要提供了关于网络资源的隔离，包括网络设备、IPv4和IPv6协议栈、IP路由表、防火墙、/proc/net目录、/sys/class/net目录、socket等。

Network namespace 可以让每个容器拥有自己独立的网络设备（虚拟的），而且容器内的应用可以绑定到自己的端口，每个 namesapce 内的端口都不会互相冲突。

在宿主机上搭建网桥后，就能很方便的实现容器之间的通信，而且每个容器内的应用都可以使用相同的端口。

一个物理的网络设备最多存在于一个network namespace中，可以通过创建veth pair(虚拟网络设备对：有两端，类似管道，如果数据从一端传入另一端也能接受，反之亦然)在不同的network namespace间创建通道，以达到通信目的。

### user namespace

主要隔离了安全相关的标识符(identifier)和属性(attribute)，包括用户ID、用户组ID、root目录、key(指密钥)以及特殊权限。

通俗地讲，一个普通用户的进程通过clone()创建的新进程在新user namespace中可以拥有不同的用户和用户组。

这意味着一个进程在容器外属于一个没有特权的普通用户，但是它创建的容器进程却属于拥有所有权限的超级用户，这个技术为容器提供了极大的自由。

隔离用户和用户组。它的厉害之处在于，可以让宿主机上的一个普通用户在 namespace 里成为 0 号用户，也就是 root 用户。这样普通用户可以在容器内“随心所欲”，但是影响也仅限在容器内。

### mount namespace

隔离文件挂载点，每个进程能看到的文件系统都记录在/proc/$$/mounts里。在一个 namespace 里挂载、卸载的动作不会影响到其他 namespace。

## 举个namespace的例子
```
// 交互式运行一个容器,分配一个伪输入终端
docker run -it busybox /bin/sh
```

在容器内部使用ps可以查看进程id(可以看到有一个pid=1的/bin/sh)
```
/ # ps
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/sh
    5 root      0:00 ps
```
在宿主机中呢(这个对应的进程id是3702)
```
# ps -ef |grep busy
root      3702  3680  0 15:53 pts/0    00:00:00 docker run -it busybox /bin/sh
```
这就意味着，前面执行的/bin/sh，已经被Docker隔离在了一个跟宿主机完全不同的世界当中。

这在源码中实际上是这么一段：
```
// 调用clone，加入namespace，当前这个使用的是PID namespace
int pid = clone(main_function, stack_size, CLONE_NEWPID | SIGCHLD, NULL);
```

但有个问题，然后如果你自己只用PID namespace使用上述的clone()创建一个进程。

查看ps或top等命令时，却还是能看到所有进程，说明并没有完全隔离。

这是因为像ps、top这些命令会去读/proc文件系统，而此时你创建的隔离了pid的进程和宿主机使用的是同一个/proc文件系统，所以这些命令显示的东西都是一样的。

所以，我们还需要使其它的namespace隔离(即很多时候只是单纯的一个namespace的隔离不大够)，如文件系统进行隔离。

## docker与namespace?

理解docker容器本质上是一个运行在宿主机上的一个进程，那么进程就可以使用之前的namespace手段。

创建容器时，通过namespace提供需要的系统资源给容器进程，隔离出自己单独的空间.

## cgroup(资源限额)

namespace并不能提供物理资源上的隔离，比如 CPU 或者内存，如果在同一台机器上运行了多个对彼此以及宿主机器一无所知的容器，这些容器却共同占用了宿主机器的物理资源。

如果其中的某一个容器正在执行 CPU 密集型的任务，那么就会影响其他容器中任务的性能与执行效率，导致多个容器相互影响并且抢占资源。

Linux Cgroups(Control Groups) 提供了对一组进程及将来的子进程的资源的限制 ，控制和统计的能力，这些资源包括CPU，内存，存储，网络等。

通过Cgroups，可以方便的限制某个进程的资源占用，并且可以实时的监控进程的监控和统计信息。

### 两种cgroup驱动

- systemd

system daemon 的cgroup驱动

- cgroupfs

要用 CPU share 为多少，直接把 pid 写入对应的一个 cgroup 文件，然后把对应需要限制的资源也写入相应的 memory cgroup 文件和 CPU 的 cgroup 文件。

### 三个组件

- cgroup

对进程分组管理的一种机制。一个cgroup包含一组进程，并可以在这个cgroup上增加Linux subsystem的各种参数的配置，将一组进程和一组subsystem的系统参数关联起来。亦即，管理“组进程”并关联参数

- subsystem

subsystem 是一组资源控制的模块，一般包含有：

**blkio** 设置对块设备（比如硬盘）的**输入输出**的访问控制（block/io），限制磁盘IOPS和bps速率

**cpu** 设置cgroup中的进程的**CPU**被调度的策略

cpuacct 可以统计cgroup中的进程的CPU占用(cpu account)

cpuset 在多核机器上设置cgroup中的进程可以使用的CPU和内存（此处内存仅使用于NUMA架构）

**devices** 控制cgroup中进程对**设备**的访问

**freezer** 用于**挂起**(suspends)和**恢复**(resumes) cgroup中的进程

**memory** 用于控制cgroup中进程的内存占用

net_cls 用于将cgroup中进程产生的网络包分类(classify)，以便Linux的tc(traffic controller) (net_classify) 可以根据分类(classid)区分出来自某个cgroup的包并做限流或监控。

net_prio 设置cgroup中进程产生的网络流量的优先级

ns 这个subsystem比较特殊，它的作用是cgroup中进程在新的namespace fork新进程(NEWNS)时，创建出一个新的cgroup，这个cgroup包含新的namespace中进程。

- hierarchy

hierarchy 的功能是把**一组cgroup**串成一个树状的结构，一个这样的树便是一个hierarchy，通过这种树状的结构，Cgroups可以做到继承。

比如我的系统对一组定时的任务进程通过cgroup1限制了CPU的使用率，然后其中有一个定时dump日志的进程还需要限制磁盘IO。

为了避免限制了影响到其他进程，就可以创建cgroup2继承于cgroup1并限制磁盘的IO，这样cgroup2便继承了cgroup1中的CPU的限制，并且又增加了磁盘IO的限制而不影响到cgroup1中的其他进程。

## cgroup操作实例

- 创建并挂载一个hierarchy:
```
mkdir cgroup-test
sudo mount -t cgroup -o none,name=cgroup-test cgroup-test ./cgroup-test
ls ./cgroup-test
```

- 在创建好的hierarchy上cgroup根节点扩展出两个子cgroup，子cgroup会扩展父cgroupd的属性。

- 在cgroup中添加和移动进程

一个进程在一个cgroup的hirearchy进程中，只能在一个cgroup节点上存在，只要把该进程ID写入该cgroup节点的tasks文件，就可以把进程移动到这个cgroup节点。

- 通过subsystem限制cgroup中的进程资源
```
mount | grep memory
```

查看哪个目录挂在了memory subsystem的hierarchy上,并修改limit_in_bytes

```
sudo sh -c "echo "100m" > memory.limit_in_bytes" sudo sh -c "echo "100m" > memory.limit_in_bytes"
```

## cgroup+namespace+ 在docker中实际上是有libcontainer来集成的

## rootfs

rootfs 代表一个 Docker 容器在启动时(而非运行后)其内部进程可见的文件系统视角，或者叫 Docker 容器的根目录。

先来看一下，Linux 操作系统内核启动时，内核会先挂载一个只读的 rootfs，当系统检测其完整性之后，决定是否将其切换到读写模式。

Docker 沿用这种思想，不同的是，挂载rootfs 完毕之后，没有像 Linux 那样将容器的文件系统切换到读写模式，而是利用联合挂载技术(UnionFS)，在这个只读的 rootfs 上挂载一个读写的文件系统，挂载后该读写文件系统空空如也。

Docker 文件系统简单理解为：只读的 rootfs + 可读写的文件系统。

<div align="center"> <img src="https://upload-images.jianshu.io/upload_images/13880937-537571f61796d2e9.png?imageMogr2/auto-orient/strip|imageView2/2/w/546/format/webp" width=""> </div><br>

### UnionFS

简单来说就是支持将不同目录挂载到同一个虚拟文件系统下的文件系统。

这种文件系统可以一层一层地叠加修改文件。无论底下有多少层都是只读的，只有最上层的文件系统是可写的。

当需要修改一个文件时，创建该文件的一个副本，将文件从只读层复制到可写层进行修改，结果也保存在可写层。在Docker中，底下的只读层就是image，可写层就是Container。(类比镜像和容器)

## 隔离之后如何进行通信？————docker网络相关

所以 Docker 虽然可以通过命名空间创建一个隔离的网络环境，但是 Docker 中的服务仍然需要与外界相连才能发挥作用。

每一个使用 docker run 启动的容器其实都具有单独的网络命名空间，Docker 为我们提供了四种不同的网络模式，Host、Container、None 和 Bridge 模式。(安装Docker时，它会自动创建三个网络，bridge（创建容器默认连接到此网络）、 none 、host)

run容器时使用-network参数(host/none/bridge/container:NAME_or_ID)指定连接到哪个网络

### bridge模式(默认接入该网络)

Docker 会为所有的容器设置 IP 地址。当 Docker 服务器在主机上启动之后会创建新的虚拟网桥 docker0，随后在该主机上启动的全部服务在默认情况下都与docker0相连。

在默认情况下，每一个容器在创建时都会创建一对虚拟网卡，两个虚拟网卡组成了数据的通道，其中一个会放在创建的容器中，会加入到名为 docker0 网桥中。

docker0 会为每一个容器分配一个新的 IP 地址并将 docker0 的 IP 地址设置为默认的网关。网桥 docker0 通过 iptables 中的配置与宿主机器上的网卡相连，所有符合条件的请求都会通过 iptables 转发到 docker0 并由网桥分发给对应的机器。

Docker 通过 Linux 的命名空间实现了网络的隔离，又通过 iptables 进行数据包转发，让 Docker 容器能够优雅地为宿主机器或者其他容器提供服务。

#### bridge模式拓扑

虚拟网桥的工作方式和物理交换机类似，这样主机上的所有容器就通过交换机连在了一个二层网络中。

接下来就要为容器分配IP了，Docker会选择一个和宿主机不同的IP地址和子网分配给docker0，连接到docker0的容器就从这个子网中选择一个未占用的IP使用。

如一般Docker会使用172.17.0.0/16这个网段，并将172.17.0.1/16分配给docker0网桥（在主机上使用ifconfig命令是可以看到docker0的，可以认为它是网桥的管理接口，在宿主机上作为一块虚拟网卡使用）。

单机环境下的网络拓扑如下，主机地址为10.10.0.186/24。

<div align="center"> <img src="https://img-blog.csdnimg.cn/20190702230627173.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21lbHRzbm93,size_16,color_FFFFFF,t_70" width=""> </div><br>

#### Bridge配置详解

（1）在主机上创建一对虚拟网卡veth pair设备。veth设备总是成对出现的，它们组成了一个数据的通道，数据从一个设备进入，就会从另一个设备出来。因此，veth设备常用来连接两个网络设备。

（2）Docker将veth pair设备的一端放在新创建的容器中，并命名为eth0。另一端放在主机中，以veth65f9这样类似的名字命名，并将这个网络设备加入到docker0网桥中，可以通过brctl show命令查看。

（3）从docker0子网中分配一个IP给容器使用，并设置docker0的IP地址为容器的默认网关。

#### Bridge下怎么进行容器通信

在bridge模式下，连在同一网桥上的容器可以相互通信（若出于安全考虑，也可以禁止它们之间通信，方法是在DOCKER_OPTS变量中设置–icc=false，这样只有使用–link才能使两个容器通信）。

Docker默认配置–icc=true，也就是说，宿主机上的所有容器可以不受任何限制地相互通信，这可能导致拒绝服务攻击。

进一步地，Docker可以通过–ip_forward和–iptables两个选项控制容器间、容器和外部世界的通信。

容器也可以与外部通信，我们看一下主机上的Iptable规则，可以看到这么一条
```
-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
```
这条规则会将也就是从Docker容器产生的包，并且不是从docker0网卡发出的，进行源地址转换，转换成主机网卡的地址。

举一个例子:

假设主机有一块网卡为eth0，IP地址为10.10.101.105/24，网关为10.10.101.254。

从主机上一个IP为172.17.0.1/16的容器中ping百度（180.76.3.151）。

IP包首先从容器发往自己的默认网关docker0，包到达docker0后，也就到达了主机上。然后会查询主机的路由表，发现包应该从主机的eth0发往主机的网关10.10.105.254/24。

接着包会转发给eth0，并从eth0发出去（主机的ip_forward转发应该已经打开）。

这时候，上面的Iptable规则就会起作用，对包做SNAT转换，将源地址换为eth0的地址。这样，在外界看来，这个包就是从10.10.101.105上发出来的，Docker容器对外是不可见的。

那么，外面的机器是如何访问Docker容器的服务呢？我们首先用创建一个含有web应用的容器，将容器的80端口映射到主机的80端口。

然后查看Iptable规则的变化，发现多了这样一条规则：
```
-A DOCKER ! -i docker0 -p tcp -m tcp --dport 80 -j DNAT --to-destination 172.17.0.2:80
```
此条规则就是对主机eth0收到的目的端口为80的tcp流量进行DNAT转换，将流量发往172.17.0.2:80，也就是我们上面创建的Docker容器。所以，外界只需访问10.10.101.105:80就可以访问到容器中的服务。

除此之外，我们还可以自定义Docker使用的IP地址、DNS等信息，甚至使用自己定义的网桥，但是其工作方式还是一样的。

### Host模式

与宿主机在同一个网络中，但没有独立IP地址。

如果启动容器的时候使用host模式，那么这个容器将不会获得一个独立的Network Namespace，而是和宿主机共用一个Network Namespace。

容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。

### container模式(类比host)

这个模式指定新创建的容器和已经存在的一个容器共享一个Network Namespace，而不是和宿主机共享。

新创建的容器不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。两个容器的进程可以通过lo网卡设备通信。

### none模式

该模式将容器放置在它自己的网络栈中，但是并不进行任何配置。

实际上，该模式关闭了容器的网络功能，在以下情况下是有用的：容器并不需要网络（例如只需要写磁盘卷的批处理任务）。

### 自定义网络
```
docker network create --driver bridge new_bridge
```

## Docker怎么建立image，创建容器？

1. image建立

<div align="center"> <img src="https://img-blog.csdnimg.cn/20190927222252368.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zODQ5OTIxNQ==,size_16,color_FFFFFF,t_70" width=""> </div><br>


- docker镜像由一层层layer组合起来

- 每一层layer对应的是Dockerfile中的一条指令

- 这些layer中，只有一层layer(container layer)为R/W，layer即

- 其他layer是read-only layer,均为image layer(R/O)

- 通常最后一层为CMD层或者ENTRYPOINT层，这一层是R/W的

所有的容器其实都是共用一个image.镜像是只读的,每一层都是只读的.

在内核的上面,最底层是一个基础镜像层(Debian),如果想要在ubuntu基础镜像上装一个emacs编辑器,则只能在基础镜像之上,再去构建一个新的镜像层.

以此类推,如果想要在emacs镜像层上添加一个apache,则只能再在其上面再构建一个新的镜像层.每一层实际上都是挂载在宿主机相应的目录下.

### 那么怎么做读写？

其实在镜像的最上层,还有一个读写层,在容器启动的时候为当前容器单独挂载.所有对容器的操作都发生在读写层.一旦容器销毁,这个读写层也会随之销毁.

所以 容器 = 镜像 + 读写层

对这个读写层的操作主要有写时复制和用时配置。

- 写时复制

所有的文件与数据都是从image读取.当要对文件进行写操作的时候,才从image把要写的文件复制到容器内的文件系统进行修改.

所以无论有多少个容器共享同一个image,所做的写操作其实都是对从image中复制到自己的文件系统中的复本中进行,不会修改image的源文件.

- 用时配置

启动一个新的容器的时候,并不会为这个容器预分配一些磁盘空间,而是当有新文件写入的时候,才会按需分配新空间.

2. 创建容器

- 先创建容器,然后配置Namespace,创建父进程.

- 挂载文件系统然后替换init进程为用户进程

- 完成容器创建

## 容器引擎

<div align="center"> <img src="https://segmentfault.com/img/bVbgQoC?w=639&h=285
" width=""> </div><br>

### Docker Daemon

作为Docker容器管理的守护进程.守护进程模块不停的在重构，其基本功能和定位没有变化。和一般的CS架构系统一样，守护进程负责和Docker client交互，并管理Docker镜像、容器。

### Containerd

<div align="center"> <img src="https://upload-images.jianshu.io/upload_images/8911567-132e0c0087881517.png?imageMogr2/auto-orient/strip|imageView2/2/w/862/format/webp" width=""> </div><br>

containerd主要职责是镜像管理（镜像、元信息等）、容器执行（调用最终运行时组件执行）。独立负责容器运行时和生命周期（如创建、启动、停止、中止、信号处理、删除等）。

containerd向上为Docker Daemon提供了gRPC接口，使得Docker Daemon屏蔽下面的结构变化，确保原有接口向下兼容。

向下通过containerd-shim结合runC，使得引擎可以**独立升级**，避免之前Docker Daemon升级会导致所有容器不可用的问题。

### docker-shim

docker-shim是一个真实运行的容器的真实垫片载体，每启动一个容器都会起一个新的docker-shim的一个进程，

他直接通过指定的三个参数：

容器id，boundle目录（containerd的对应某个容器生成的目录，一般位于：/var/run/docker/libcontainerd/containerID），运行时二进制（默认为runc）

来调用runc的api创建一个容器。

### RunC

runC基于libcontainer库，是用来运行容器的。

Docker默认提供了docker-runc实现，事实上，通过containerd的封装，可以在Docker Daemon启动的时候指定runc的实现。

### 实例

首先，需要创建容器标准包，这部分实际上由containerd的bundle模块实现，将Docker镜像转换成容器标准包。
```
docker export $(docker create busybox) | tar -C rootfs -xvf -
```
上述命令将busybox镜像解压缩到指定的rootfs目录中。如果本地不存在busybox镜像，containerd还会通过distribution模块去远程仓库拉取。

此时，准备所需的文件，接下来我们需要创建配置文件：
```
docker-runc spec
```
此时会生成一个名为config.json的配置文件，该文件和Docker容器的配置文件类似，主要包含容器挂载信息、平台信息、进程信息等容器启动依赖的所有数据。
```
runc run busybox
或者
docker-runc run busybox
```
执行之后，我们可以看见容器已经启动：

此时，事实上已经可以不依赖Docker本身，如果系统上安装了runc包，即可运行容器。

从这里可以看到标准化的重要性。
