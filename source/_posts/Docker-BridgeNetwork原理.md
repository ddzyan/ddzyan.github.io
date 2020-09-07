---
title: Docker-BridgeNetwork原理
date: 2020-09-07 10:21:36
categories:
  - docker
tag:
  - docker
---

## 参考资料

- [三层交换机](https://blog.csdn.net/tshangshi/article/details/50894478)
- [OSI 网络架构](https://zhuanlan.zhihu.com/p/32059190)
- [七层网络架构](https://www.lifewire.com/layers-of-the-osi-model-illustrated-818017)

## 疑问

- 为什么已经有了链路层的交换机，还要有网络层的路由器呢?
- 为什么主机之间需要 MAC 地址才能进行通信？
- 有了 MAC 地址，为什么不直接通过 MAC : prot 进行发送数据？
- 简述在 docker 中被隔离的 container 如何与其他 Network Namespace 里的 container 进行交互？
- docker 使用宿主机网络栈（host 模式）会引起什么问题？

## 基础概念（可以先跳过）

### 操作指令

```shell
# 查看路由表
$ route

# 查看网络桥接连接信息
$ brctl show

# 查看网络接口信息
$ ip link show

# 查看网络接口绑定信息
$ cat /sys/class/net/etho/iflnk

# 显示数据包到主机的路径
$ traceroute

# 显示 IP 对应的 mac 地址0
$ ip neigh show dev cali33e0eb4cdea
10.233.96.1 lladdr aa:7a:7e:d0:be:fa REACHABLE
```

<!--more-->

### 网络栈

每个被隔离的 container 都有一套网络栈包含：包含网卡（Network Interface），回环设备，路由表（Routing Table）和 iptables 规则。

### 网桥

网桥是工作在 **数据链路层** 的设备，主要功能是根据 MAC 地址来将数据包转发到网桥的不同 Port 上。

### 交换机工作原理

当交换机接收到一个数据帧，提取出该数据帧的目的 MAC 地址，并依此为根据进行 CAM 表查询，如果能查找到结果，则根据结果进行数据帧的转发，如果不能命中则（向除接收端口外的）所有端口进行 UDP 广播 。如果有端口匹配的 MAC 地址匹配，则会进行响应，交换机会将该 MAC 地址和接收到该 MAC 地址的端口绑定起来，插入 CAM 表项，这样当接收到一个发送到该 MAC 地址的数据帧时，就不需要向所有端口广播。

二层设备只认帧中的源和目的 MAC 地址进行数据传输，当传到第三层路由器，三层设备却是基于数据报文中的 IP 进行数据转发。

在 OSI 网络模型中，每层都是对上一层的数据包包装并且加入自己的内容，反之则对每层的数据进行解包，流程如下：

![img](https://pic1.zhimg.com/80/b6ae78d0dfafa7b3f0b66848e40cc9ce_720w.jpg?source=1940ef5c)

## docker 网络

首先需要了解，docker 内置的三种网络架构

```shell
$ docker network ls
5a6eb0dbf210        bridge              bridge              local
06e92e3d36fc        host                host                local
325b6c89555c        none                null                local
```

- bridge 桥接模式
- host 直连使用宿主机网络栈，不开启 Network Namespace
- none 不进行设置，网络为隔离状态

### Bridge Network

docker 会默认在宿主机上创建一个 docker0 网桥，实现连接在 docker0 网桥上的容器通讯。而容器与 docker0 网桥的连接需要通过 Veth Pire 虚拟设备，它的特点是：总是成对出现，并且会将一头网卡的数据包转发到另一头网卡，即使这两个网卡在不同的 Network Namespace。

docker 使用 Veth Pire 和 docker0 组成三层交换机，实现数据传输：

1. 物理层 --- veth pire
2. 链路层 --- 网桥 docker0 --- 交换机
3. 网络层 --- 根据 IP 地址进行通讯 --- 路由器

## 测试容器网络连接

创建用于测试的 2 个不同 network namespace 的 container，查看 2 个 container 的 ip 地址，并且进行 ping 验证是否能连接

```shell
# 创建容器
$ docker run -d --name busybox-1 busybox tail -f
$ docker run -d --name busybox-2 busybox tail -f

# 查看 container ip 地址
$ docker inspect bridge
  "Containers": {
            "2942d0f9f19de0eda505d8da0c7a6be0ddffc0e35059426578e0465e7cb00a91": {
                "Name": "busybox-2",
                "EndpointID": "16c3871e1470cb0924297e33acc7c6ba2cda8366673a2daf73d2862d7e8c4b1e",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "cb5f1945a02f28074367c07b641b0291f29d5e854add13f12960930952cc390e": {
                "Name": "busybox-1",
                "EndpointID": "37b30e6b22463208edf0de8458a55218712ff4752413cded81e7b8e9c28976e2",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },

# 验证网络连接
$ docker exec busybox-1 ping 172.17.0.3
PING 172.17.0.3 (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.456 ms
64 bytes from 172.17.0.3: seq=1 ttl=64 time=0.064 ms
64 bytes from 172.17.0.3: seq=2 ttl=64 time=0.059 ms
```

### 宿主机网络

以下宿主机输出内容去除了部分无关信息，发现宿主机上多了 docker0 网桥和两个 Veth Pire 虚拟设备，并且通过指令可以看到 2 个虚拟设备已经和 docker0 完成连接

```shell
$ ifconfig
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:19ff:feef:cd35  prefixlen 64  scopeid 0x20<link>
        ether 02:42:19:ef:cd:35  txqueuelen 0  (Ethernet)
        RX packets 1  bytes 28 (28.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 5  bytes 446 (446.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth99590bf: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::d0b1:feff:fec3:dcd3  prefixlen 64  scopeid 0x20<link>
        ether d2:b1:fe:c3:dc:d3  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 15  bytes 1242 (1.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vethb91c825: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::d836:cbff:fe89:985a  prefixlen 64  scopeid 0x20<link>
        ether da:36:cb:89:98:5a  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 14  bytes 1152 (1.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0


$ apt install bridge-utils

# 查看网桥信息一
bridge name	bridge id		STP enabled	interfaces
docker0		8000.0242f88d9fbd	no		veth99590bf
							            vethb91c825

# 查看网络连接方式二
$ ip link show
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
149: veth5b40cb5@if148: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether 4e:06:30:16:4a:0e brd ff:ff:ff:ff:ff:ff link-netnsid 2
151: veth099d3cd@if150: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether 2a:a6:6d:61:a1:fe brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

### 容器网络

进入 container busybox-1 查看网络信息，可以看到 container 的 etho 网卡是连接在宿主机上的 149 veth pire 虚拟设备上，也就是说通过 etho 的数据包最终会发到宿主机的 docker0 上进行处理

```shell
$ ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:04
          inet addr:172.17.0.4  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:11 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:866 (866.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

$ cat /sys/class/net/eth0/iflink
149

$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         172.17.0.1      0.0.0.0         UG    0      0        0 eth0
172.17.0.0      *               255.255.0.0     U     0      0        0 eth0
```

进入 container busybox-2 查看网络，发现结果也一样

```shell
$ ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:11 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:866 (866.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

$ cat /sys/class/net/eth0/iflink
151
```

### 网络架构

![img](https://static001.geekbang.org/resource/image/e0/66/e0d28e0371f93af619e91a86eda99a66.png)

## 简述单机容器网络数据包发送流程

容器之间如果需要进行网络通讯，则需要拿到目标地址的 ip 和 MAC 地址

1.  busybox-1 通过内部的 etho 网卡发送一个 ARP 广播，数据包（包含目标地址 IP）通过二层网络到达宿主机 docker0 网卡进行广播，而 busybox-2 也是 eth0 网卡也通过 veth pire 连接在 docker0 上，所以收到消息后将 172.17.0.2 的 MAC 地址回复给 busybox-1。
2.  有了 MAC 地址后，busybox-1 就会将数据包通过 eth0 网卡发送到 docker0 上
3.  docker0 根据它的 CAM 表查到对应端口为：veth099d3cd，然后把数据包发往这个端口，此端口会将接收的数据发送到 busybox-2 内的 eth0 网卡

在宿主机上查看网络转发日志，验证数据转发流程

```shell
$ iptables -t raw -A OUTPUT -p icmp -j TRACE
$ iptables -t raw -A PREROUTING -p icmp -j TRACE
$ cat /var/log/kern.log
Sep  4 02:39:10 k8s-master kernel: [ 1293.325796] TRACE: filter:FORWARD:rule:1 IN=docker0 OUT=docker0 PHYSIN=vethb91c825 PHYSOUT=veth99590bf MAC=02:42:ac:11:00:02:02:42:ac:11:00:03:08:00 SRC=172.17.0.3 DST=172.17.0.2 LEN=84 TOS=0x00 PREC=0x00 TTL=64 ID=15820 PROTO=ICMP TYPE=0 CODE=0 ID=1536 SEQ=504
```
