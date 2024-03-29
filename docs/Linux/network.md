title: Network

# **Network**

## **Utils & tools**

#### **ip commands**

10 Useful IP commands:

* `ip addr show` --- show all IP Addresses of all interfaces.
* `ip addr add 192.168.50.5 dev eth1` --- assign a IP Address to a specific interface.
* `ip addr del 192.168.50.5/24 dev eth1` --- remove an IP Address.
* `ip link set eth1 up` --- enable a network interface.
* `ip link set eth1 down` --- disable a network interface.
* `ip route show` --- show route table.
* `ip route add 10.10.20.0/24 via 192.168.50.100 dev eth0` --- add a static route.
* `ip route del 10.10.20.0/24` --- remove a static route.
* `ip route add default via 192.168.50.100` --- add default gateway.


#### **ipftrace2**

[ipftrace2](https://github.com/YutaroHayakawa/ipftrace2) is a tool which allows you to trace the journey of
packets inside the Linux kernel




--------------------------------------------------------------------------------------------------------------
## **Container & Kubernetes Network**

[A great explanation](https://learnk8s.io/kubernetes-network-packets) of how networking is done in k8s clusters.
