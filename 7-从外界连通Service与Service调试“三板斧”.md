使用NodePort可以从外部访问到Kubernetes里创建的Service。

kube-proxy要做的，只是在每台宿主机添加iptable规则。对IP包，进行一次SNAT操作，把IP源地址替换成该宿主机的CNI网桥地址（不存在CNI网桥则为宿主机IP）

为什么要做SNAT操作？（）

如果需要知道源IP，可以设置，这样一台宿主机的iptables规则，会设置为只将IP包转发给运行在这台宿主机的Pod。

第三种方式，是Kubernetes在1.7之后支持的，叫ExternalName

Kubernetes里面的Service和DNS机制，都不具备多租户能力。
