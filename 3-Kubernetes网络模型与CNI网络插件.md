CNI网桥和docker0网桥功能几乎一样，单独设置了一个，主要是：

1.Kubernets项目没有使用Docker的网络模型（CNM），所以不具体配置docker0网桥的能力。

2.与如何配置Pod，也就是Infra容器的Network Namespace有关

CNI的设计思想，就是：Kubernetes在启动Infra容器之后，可以直接调用CNI网络插件，为这个Infra容器的Network Namespace配置符合预测的网络栈。

当Kubelet组件需要创建Pod的时候，第一个创建的一定是Infra容器。所以在这下一步，dockershim会先调用Docker API创建并启动Infra容器，然后执行SetUpPod方法。该方法作为是为CNI插件准备参数，然后调用CNI插件为Infra容器配置网络。
ADD和DEL操作，是CNI插件唯一要实现的两个方法。意味着把容器以Veth Pair的方式插到CNI网桥上，或者从网桥上拔掉。

ADD操作需要的参数包括容器里网卡的名字eth0、Pod的Network Namespace文件路径（/proc/<容器进程的PID>/ns/net）、容器ID等

首先CNI bridge插件会在宿主机上检查CNI网桥是否存在，如果没有就创建。然后通过Infra容器的Network Namespace文件，进入到这个Network Namespace，然后创建一对Veth Pair设备。连接之后，CNI bridge插件还会为它设置Haripin Mode（发夹模式），这样就允许一个数据包从一个端口进来后，再从这个端口发出去（比如Pod里通过自己的Service访问到自己）。

接下来，CNI bridge插件调用CNI ipam插件，从ipam.sbnet字段规定的网段里为容器分配一个可用的IP地址，然后CNI bridge插件就把这个IP地址添加在容器的eth0网卡上，同时为容器设置默认路由。

最后，CNI bridge为CNI网桥添加IP地址。

然后CNI插件把容器的IP地址等信息返回给dockershim，然后被kubelet添加到Pod的Status字段。
