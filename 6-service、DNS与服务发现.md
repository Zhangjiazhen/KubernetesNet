可以复用k8s的service提VIP功能

Service是由kube-proxy组件，加上iptables共同实现的。

但是当宿主有大量Pod时，iptables规则需要不断刷新，会大量占用cpu资源。所以基于iptables的Service实现，都是制约Kubernetes项目承载更多量级的Pod的主要障碍。

IPVS模式的Service可以解决这一问题。

原理跟iptables类似，在创建Service之后，创建一个虚拟网卡（kube-ipvs0），并分配Service VIP作为IP。然后kube-proxy通过Linux的IPVS模块，为IP设置三个IPVS虚拟主机，并设置使用轮询模式(rr)作为负载均衡策略。相比于iptables，IPVS在内核的实现也是基于Netfilter的NAT模式，把过滤规则放到了内核态，极大降低维护规则的代价。

注意：IPVS只负责负载均衡和代理功能，完整的Service流程正常工作所需要的包过滤、SNAT等操作，还是要靠iptable来实现，只不过iptable规则数量有限。

Headless Service可以提供一个Pod的稳定的DNS名字，并且名字可以通过Pod名字和Service名字拼接出来。
