host-gw模式的工作原理，就是将每个Flannel子网的下一跳，设置成该子网对应的宿主机的IP地址。

Flannel子网和主机的信息，都是保存在Etcd当中。

性能损失大约在10%，其他所有基于VXLAN隧道机制的网络方案，性能损失在20%~30%。

Flannel host-gw模式必须要求集群宿主机之间是二层连通的。

Calico是和host-gw几乎完全一样的网络解决方案，核心是为每个容器的IP地址，找到它所对应的、“下一跳”的网关。

不同的是，Calico使用BGP（Border Gateway Protocol边界网关协议）自动在整个集群中分发路由信息。

BGP就是在大规模网络中实现节点路由信息共享的一种协议。

每个边界网关都运行着一个小程序，会将各自的路由表信息，通过TCP传输给其他的边界网关。其它边界的网关，将需要的信息添加到自己的路由表里。

另一个不同点是，不会在宿主机上创建任何网络设备。

主要由三部分组成：

1.Calico的CNI插件。

2.Felix。是一个DaemonSet，负责在宿主上插入路由规则。

3.BIRD。BGP客户端，负责在集群里分发路由信息。

Calico在默认配置下，是”Node-toNode Mesh"的模式，由于交换信息随节点数量增加，连接数以（N的平方）快速增长。只推荐用于少于100个节点的集群规模，更大规模使用Route Reflector模式。

Route Reflector指定几个专门的节点，负责跟所有节点建立BGP连接，学习全局的路由规则。其它节点只需要跟这几个专门的节点交换路由信息即可。

对于跨子网的IP通信，需要为Calico打开IPIP模式，使用IP隧道（IP tunnel）设备，IP包在进入IP隧道设备之后，就会被Linux内核的IPIP驱动接管。IPIP驱动会将这个IP包直接封装在一个宿主机网络的IP包中。

使用这个模式，会因为额外的封包解包导致网络性能下降。与Flanel VXLAN模式的性能大致相当。

为什么不能直接路由？公有云环境下，宿主机之间的网关，不允许用户进行干预和设置。

私有云环境可以将宿主机网关也加入到BGP Mesh里避免使用IPIP。

Calico提供了两种解决方案：

第一种，所有宿主机都跟宿主机网关建立BGP Peer关系。要求宿主机支持Dynamic Neighbors的BGP配置方式，直接跟一个网段的主机建立BGP Peer关系。

第二种，使用多个独立组件负责搜集所有路由信息，然后通过BGP协议同步给网关。

独立组件的工作原理，只需要WATCH Etcd里的宿主机和对应网段的变化信息，通过BGP协议分发给网关即可。

