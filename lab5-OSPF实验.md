---
title: 【计网实验】lab5-OSPF路由实验 
date: 2025-12-07 19:37:18
tags: 学习经验
categories:
- [学习经验, 计网实验]
math: true
---
1. OSPF协议概述及基本配置
2. OSPF协议报文交互过程分析
3. OSPF协议链路状态描述
4. 区域划分及LSA种类
5. OSPF协议的路由计算
6. 设计型实验1
7. 设计型实验2

关于共享文档的使用
1. 共同编辑需要登录账号
2. 遇到错误的代码/步骤欢迎直接进行修改，如果有不确定的也可以通过评论指出来~~
3. 从本次开始，将在笔记的最后开一个讨论区，欢迎来讨论各种各样的事情，比如本次计网实验体验如何/难度如何，甚至可以不止计网~？
4. 如果有更好用的markdown共享网站/更多的建议欢迎在评论区留言——
## 1 mooc

![12](https://hackmd.io/_uploads/BJGxmWwMZx.png)
![13](https://hackmd.io/_uploads/HJugmZDMWe.png)


## 2 OSPF 协议概述及基本配置

### 2.1 实验步骤

###### 1 组网

![2](https://hackmd.io/_uploads/H1bWmZvG-l.png)


##### 2 配置

tips:或许本次实验所有有关 router id 的配置都要在前面加上（存疑）
``` plaintext
interface LoopBack 1
ip address 1.1.1.1 255.255.255.255
quit
```

1. R1 配置
```plaintext
undo interface vlan 1

sys
sysn R1
# 配置router id
router id 1.1.1.1
# 配置E0/0接口
interface Ethernet 0/0
ip address 168.1.1.1 255.255.255.0
quit
# 配置OSPF
ospf
area 0
network 1.1.1.0 0.0.0.255
network 168.1.1.0 0.0.0.255
```

2. R2 配置

```plaintext
undo interface vlan 1

sys
sysn R2
# 配置router id
router id 2.2.2.2
# 配置E0/0接口
interface Ethernet 0/0
ip address 168.1.1.2 255.255.255.0
quit
# 配置OSPF
ospf
area 0
network 2.2.2.0 0.0.0.255
network 168.1.1.0 0.0.0.255
```

##### 3

**截图**
```plaintext
# 检查两个路由器中的IP路由表，看有没有对方的OSPF，如
#    2.2.2.2/32     OSPF    10   1           168.1.1.2       Ethernet0/0
display ip routing-table

# 查看R2的OSPF邻接信息，看到如
# 2.2.2.2 168.1.1.2 1 36 Eth0/0 Full/BDR
dis ospf peer
```

##### 4

```plaintext
# 更改R1的router id
undo router id
router id 3.3.3.3

# 显示OSPF的概要信息（如果生效，能看到RouterID:3.3.3.3）
display ospf brief

# 如果没生效
reset ospf process # 或reset ospf all

# 最后需要把R1的router id改回去（存疑）
```

### 2.2 实验报告
1． 查看R2的OSPF的邻接信息，写出其命令和显示的结果：
命令：display ospf peer

2． 将R1的router id 更改为3.3.3.3，写出其命令。显示OSPF的概要信息，查看此更改是否生效。如果没有生效，如何使其生效？
undo router id
router id 3.3.3.3
dis ospf brief
如果没有生效，进行重启：reset ospf process

## 3. OSPF协议报文交互过程分析

实验内容：在路由器上启动OSPF协议，同时在计算机上运行Wireshark软件截取报文，分析OSPF协议的报文结构、邻居状态机和报文交互过程。

### 实验步骤

#### 3.6.1 OSPF协议报文格式

步骤1 **继续**上一节的实验（上一实验配置保留，组网相同），在每台计算机上运行Wireshark软件，开始捕获报文。用PCC监听路由器R1 E0/0接口的报文。

步骤2 将交换机与路由器的两根线断开，然后再快速地将两根线重新连接。使两台路由器同时重新启动OSPF协议

（截图）

保存截获报文。

步骤3 分析截获的报文，可以看到OSPF的5种协议报文，请写出这5种协议报文的名称。（实验报告）

分别为：Hello Packet，DB Description，LS Request，LS Update，LS Acknowledge。

并选择一条Hello报文

（每种都截图，展开首部和报文体所有内容）

写出整个报文的结构（OSPF首部及Hello报文体）。（实验报告）

步骤4 分析OSPF协议的头部，OSPF协议中Router ID的作用是什么？它是如何产生的？（实验报告）

步骤5 分析截获的一条LS Update报文，写出该报文的首部，并写出该报文中有几条LSA。（实验报告）

#### 3.6.2 OSPF报文交互过程

步骤1 结合截获的报文和DD报文中的字段（MS, I, M），写出DD主从关系的协商过程和协商结果。（实验报告）

步骤2 结合截获的报文和DD报文中的字段（MS, I, M, Seq），写出LSA摘要信息交互的过程，并描述其隐含确认与可靠传输机制是如何起作用的。（实验报告）

步骤3 结合截获的一组相关的LSR、LSU和LS Ack报文，具体描述OSPF协议报文交互过程中确保可靠传输的机制。（实验报告）

#### 3.6.3 邻居状态机

步骤1 在R1（PCC）上执行下列命令，显示OSPF调试信息。

```
<R1>debugging ospf event
<R1>terminal debugging
```

步骤2 断开S1与R1、R2的两根连线，然后再重新连接。

（截图保留debug信息）

请根据debug显示信息，画出R1上的OSPF邻居状态转移图。（实验报告）

## 4 OSPF 协议链路状态描述

### 4.1 实验步骤

#### 4.1.1 Router-LSA
##### 1
1. 在上个实验截取的报文中找到 LSU(Update) 报文
2. 观察发现Update报文包括两种：router-LSA和Network-LSA（LS Type: Network-LSA）

##### 2 
1. 分析报文头。展开 Open Shortest Path First - OSPF Header
2. 分析报文主体部分。展开 Open Shortest Path First - LS Update Packet - LS Type: Router-LSA，填表

#### 4.1.2 Network-LSA

##### 1 
分析报文头。展开 Open Shortest Path First - OSPF Header

##### 2 
R2 上 `display ospf lsdb network`

#### 4.1.3 指定路由器 DR 选举

##### 1 组网&配置

![3](https://hackmd.io/_uploads/HyZHQZDGWg.png)

依旧 PCC 或 PCD（？）的 Eth0 监听 R1 E0/0

1. S1 配置
```plaintext
undo interface vlan 1

sys
sysn S1
# 配置router id
router id 5.5.5.5
# 配置vlan1
vlan 1
port E1/0/1
port E1/0/2
port E1/0/13
interface Vlan-interface 1
ip address 168.1.1.3 255.255.255.0
quit
# 配置OSPF（Area 0）
ospf
area 0
network 5.5.5.0 0.0.0.255  # 宣告Loopback1
network 168.1.1.0 0.0.0.255  # 宣告直连网段
```

2. S2 配置

```plaintext
undo interface vlan 1

sys
sysn S2
# 配置Router ID
router id 6.6.6.6
# 配置vlan1
vlan 1
port E1/0/1
interface Vlan-interface 1
ip address 168.1.1.4 255.255.255.0
quit
# 配置OSPF（Area 0）
ospf
area 0
network 6.6.6.0 0.0.0.255  # 宣告Loopback1
network 168.1.1.0 0.0.0.255  # 宣告直连网段
```

##### 2

1. 查看路由表，R1,R2,S1,S2 都 `display ip routing-table`，能看到
	- 自身的 Loopback 路由（如 R2 的 2.2.2.2/32）是 `Direct` 类型（直连）；
	- 其他三台设备的 Loopback 路由都是 `OSPF` 类型（通过 OSPF 协议学习）；
	- 168.1.1.0/24 都是 `Direct` 类型（所有设备都在同一广播域，直连该网段）。
2. 查看 DR 和 BDR，以及各台设备的 Router ID 和优先级
	R2 上 `display ospf interface`，能看到
	DR为168.1.1.2，优先级1 router id 为2.2.2.2
	BDR为168.1.1.1，优先级1 router id 为 1.1.1.1

##### 3

1. R2 上 `reset ospf process`，再 `dis ospf peer`
2. DBR 应该是 6.6.6.6

#### 4.1.4 邻居状态机

##### 1
R1 上
```plaintext
<R1>debugging ospf event
<R1>terminal debugging
```

##### 2
断开 S1 与 S2、R1、R2 的连线,然后再重新连接。请根据 debug 显示信息,画出 R1 上所有 OSPF 邻居路由器的邻居状态转移图。


### 4.2 实验报告
10． 请写出图中的网络有几种网络类型？R2发出的所有Update报文中共包含几种类型的LSA，具体类型是什么？
图中的网络有两种类型，包括广播类型和点到点PTP类型。
Update报文包括两种：router-LSA和Network-LSA

11． ？
12． ？

13． 在4.6.3节步骤2中，请写出此时这个广播网络的DR和BDR，以及各台设备的Router ID和优先级，写出查看这些信息的命令。并解释为什么？

DR是168.1.1.2（优先级1 router id 为2.2.2.2）
BDR是168.1.1.1（优先级1 router id 为1.1.1.1）
Display ospf interface
因为两个的优先级相同，比较Router id ，第一大的为DR，第二大的为BDR。

14． ？
15． ？

## 5. 区域划分及LSA种类

实验内容：通过配置多区域的OSPF网络，在路由器上查看所生成的LSDB，分析常见的3~5类LSA的结构。理解OSPF路由在区域间的传播方式，以及区域外路由在OSPF协议中的引入与传播方法。

### 5.5 实验组网

```
S1 E1/0/1 --- E0/0 R1
R1 S1/0 --- S1/0 R2
R2 E0/0 --- E1/0/1 S2
```

### 5.6 实验步骤

步骤1 每4人一组，每人负责配置一个设备。按照组网要求连接各实验设备，并正确配置IP地址和Router ID。

配置S1

```
<H3C>system-view
[H3C]sysname S1
[S1]vlan 2
[S1-vlan2]port e 1/0/1
[S1-vlan2]quit
[S1]inter vlan 2
[S1-Vlan-interface2]ip address 192.168.1.1 255.255.255.0
[S1-Vlan-interface2]quit
[S1]interface loop 1
[S1-LoopBack1]ip address 4.4.4.4 255.255.255.255
```

配置R1 1.1.1.1

```
<H3C>system-view
[H3C]sysname R1
[R1]interface e 0/0
[R1-Ethernet0/0]ip address 192.168.1.2 24
[R1-Ethernet0/0]quit
[R1]interface s 1/0
[R1-Serial1/0]ip address 10.1.1.1 24
[R1-Serial1/0]shutdown
[R1-Serial1/0]undo shutdown
[R1-Serial1/0]interface loop 1
[R1-LoopBack1]ip address 5.5.5.5 32
```

配置S2 3.3.3.3

```
<H3C>system-view
[H3C]sysname S2
[S2]vlan 5
[S2-vlan5]port e 1/0/1
[S2-vlan5]quit
[S2]inter vlan 5
[S2-Vlan-interface5]ip address 20.1.1.2 255.255.255.0
[S2-Vlan-interface2]quit
[S2]interface loop 1
[S2-LoopBack1]ip address 6.6.6.6 255.255.255.255
```

配置R2 2.2.2.2

```
<H3C>system-view
[H3C]sysname R2
[R2]interface e 0/0
[R2-Ethernet0/0]ip address 20.1.1.1 24
[R2-Ethernet0/0]quit
[R2]interface s 1/0
[R2-Serial1/0]ip address 10.1.1.2 24
[R2-Serial1/0]shutdown
[R2-Serial1/0]undo shutdown
```

步骤2 在R1、R2、S2上启动OSPF协议，并在接口上指定相应的区域。

S2操作

```
[S2-LoopBack1]quit
[S2]router id 3.3.3.3
[S2]ospf
[S2-ospf]area 1
[S2-ospf-area-0.0.0.1]network 20.1.1.0 0.0.0.255
[S2-ospf-area-0.0.0.1]network 6.6.6.6 0.0.0.255
```

R1操作

```
[R1-LoopBack1]quit
[R1]router id 1.1.1.1
[R1]ospf
[R1-ospf-1]area 0
[R1-ospf-1-area-0.0.0.0]network 10.1.1.0 0.0.0.255
[R1-ospf-1-area-0.0.0.0]network 5.5.5.5 0.0.0.255
```

R2操作

```
[R2-Serial1/0]quit
[R2]router id 2.2.2.2
[R2]ospf
[R2-ospf-1]area 0
[R2-ospf-1-area-0.0.0.0]network 10.1.1.0 0.0.0.255
[R2-ospf-1-area-0.0.0.0]quit
[R2-ospf-1]area 1
[R2-ospf-1-area-0.0.0.1]network 20.1.1.0 0.0.0.255
```

步骤3 在R1上察看LSA：在完成上述配置后，当各路由器之间成功建立邻居关系并交换路由信息之后（观察路由表，如果出现了OSPF路由，则表明上述过程已经结束），就可以在路由器上用`display ospf lsdb`等命令查看LSA信息。

在R1上

```
[R1-ospf-1-area-0.0.0.0]quit
[R1-ospf-1]quit
[R1]display ospf lsdb
```

（截图）

步骤4 三类LSA的分析

```
[R1]display ospf lsdb summary
```

（截图）

步骤5 四类和五类LSA分析

配置R1

```
[R1]ip route-static 4.4.4.0 255.255.255.0 192.168.1.1
[R1]ospf
[R1-ospf]import-route static
```

配置S1

```
[S1-LoopBack1]quit
[S1]ip route 0.0.0.0 0.0.0.0 192.168.1.2
```

在R2/S2上查看路由表，这里查看R2的路由表

```
[R2-ospf-1-area-0.0.0.1]quit
[R2-ospf-1]quit
[R2]display ip routing
```

（截图）找到一条到4.4.4.0/24的OSPF_ASE路由

在R1上查看lsdb

```
[R1-ospf]quit
[R1]display ospf lsdb
```

（考虑截图）

在R2上查看四类LSA

```
[R2]display ospf lsdb asbr
```

（截图）

在R2上查看五类LSA

```
[R2]display ospf lsdb ase
```

（截图）

## 6 OSPF 协议的路由计算

### 6.1 实验步骤

##### 1 组网&配置 router ID, IP 地址

![4](https://hackmd.io/_uploads/SJImmbwf-x.png)


1. S1
```plaintext
undo interface vlan 1

sys
sysname S1

vlan 2
port e 1/0/24
inter vlan 2
ip add 30.1.1.2 24
quit

vlan 3
port e 1/0/1
inter vlan 3
ip add 40.1.1.1 24
quit

router id 1.1.1.1
```

2. S2
```plaintext
undo interface vlan 1

sys
sysname S2

vlan 2
port e 1/0/1
inter vlan 2
ip add 10.1.1.2 24
quit

router id 4.4.4.4
```

3. R1
```plaintext
undo interface vlan 1

sys
sysname R1

inter e 0/0
ip add 40.1.1.2 24
quit

inter Serial 1/0
ip add 20.1.1.2 24
shutdown # 不确定这玩意有啥用
undo shutdown # 不确定这玩意有啥用
quit

router id 2.2.2.2
```

4. R2
```plaintext
sys
sysname R2

inter e 0/1
ip add 30.1.1.1 24
quit

inter e 0/0
ip add 10.1.1.1 24
quit

inter Serial 1/0
ip add 20.1.1.1 24
shutdown
undo shutdown

router id 3.3.3.3
```

##### 2 配置 OSPF 协议

1. S1
```plaintext
ospf
area 0
network 30.1.1.0 0.0.0.255
network 40.1.1.0 0.0.0.255
inter vlan 3
ospf cost 100
inter vlan 2
ospf cost 200
```

2. S2
```plaintext
ospf
area 0
network 10.1.1.0 0.0.0.255

inter vlan 3
ospf cost 300
```

3. R1
```plaintext
ospf
area 0
network 40.1.1.0 0.0.0.255
network 20.1.1.0 0.0.0.255
quit
quit

inter e 0/0
ospf cost 100
quit

inter Serial 1/0
ospf cost 500
```

4. R2
```plaintext
ospf
area 0
network 10.1.1.0 0.0.0.255
network 20.1.1.0 0.0.0.255
network 30.1.1.0 0.0.0.255
quit
quit

inter e 0/1
ospf cost 200
quit

inter e 0/0
ospf cost 300
quit

inter Serial 1/0
ospf cost 500
```

##### 3 ping 一 ping

1. 在 S1 上测试
	- 测试与 R1 连通：`ping 40.1.1.2`（S1 的 VLAN3 和 R1 的 E0/0 直连）
	- 测试与 R2 连通：`ping 30.1.1.1`（S1 的 VLAN2 和 R2 的 E0/1 直连）
	- 测试与 S2 连通：`ping 10.1.1.2`（需 OSPF 学习路由）
2. 在 S2 上测试
	- 测试与 R2 连通：`ping 10.1.1.1`（S2 的 VLAN2 和 R2 的 E0/0 直连）
	- 测试与 S1 连通：`ping 30.1.1.2`（需 OSPF 学习路由）
	- 测试与 R1 连通：`ping 40.1.1.2`（需 OSPF 学习路由）
3. 在 R1 上测试
	- 测试与 S1 连通：`ping 40.1.1.1`（R1 的 E0/0 和 S1 的 VLAN3 直连）
	- 测试与 R2 连通：`ping 20.1.1.1`（R1 的 S1/0 和 R2 的 S1/0 直连）
	- 测试与 S2 连通：`ping 10.1.1.2`（需 OSPF 学习路由）
4. 在 R2 上测试
	- 测试与 S1 连通：`ping 30.1.1.2`（R2 的 E0/1 和 S1 的 VLAN2 直连）
	- 测试与 S2 连通：`ping 10.1.1.2`（R2 的 E0/0 和 S2 的 VLAN2 直连）
	- 测试与 R1 连通：`ping 20.1.1.2`（R2 的 S1/0 和 R1 的 S1/0 直连）

##### 4 SPF 的计算过程分析

看书吧，还是看书吧，这段忒长了（不过是纯分析，没有实验也不用填报告(▰˘◡˘▰)）

##### 5 SPF 的计算过程分析（最短路径树计算）

看书吧，还是看书吧，这段忒长了（不过是纯分析，没有实验也不用填报告(▰˘◡˘▰)）

##### 6 思考题

R2 上
```plaintext
display ospf lsdb router
```

然后参考书上第 4、5 步，你就分析去吧（´◔ ₃ ◔ \`)


### 6.2 实验报告

21． 在6.6节的步骤2中，请参照以上配置，写出R2和S2上的配置命令 :
S2
```plaintext
ospf
area 0
network 10.1.1.0 0.0.0.255
inter vlan 3
ospf cost 300
```
R2
```plaintext
ospf
area 0
network 10.1.1.0 0.0.0.255
network 20.1.1.0 0.0.0.255
network 30.1.1.0 0.0.0.255
quit
quit
inter e 0/1
ospf cost 200
quit
inter e 0/0
ospf cost 300
quit
inter Serial 1/0
ospf cost 500
```

22． 请将以R2为根计算生成树，列出到网络中各点的下一跳以及OSPF metric值的表格，再画出相应的最短路径树。
![5](https://hackmd.io/_uploads/Byb4QWvGWe.png)


23． 请结合所做的实验思考，OSPF为什么是无自环的？（区域内、区域间）
对于区域内，OSPF使用迪杰斯特拉（Dijkstra）最短路径优先算法来计算到达所有网络的最短路径。它总是寻找到目的地的最短（即成本最低）路径，最终生成树形结构，不会产生环。而对于区域间，则主要关心ABR与各区域的关系，再集中将信息发送到骨干区域，骨干区域再发送给其他区域，保证了不会出现自环。

## 7. 设计型实验1

步骤1 实验组网，参考课本

步骤2 在S1和S2上划分Vlan

配置S1 Vlan

```
[S1]vlan 2
[S1-vlan2]port e 1/0/24
[S1-vlan2]inter vlan 2
[S1-Vlan-interface2]ip address 192.168.5.1 255.255.255.0
[S1-Vlan-interface2]quit
[S1-vlan2]quit
[S1]vlan 1
[S1-vlan1]port e 1/0/1
[S1-vlan1]inter vlan 1
[S1-Vlan-interface1]ip address 192.168.3.2 255.255.255.0
```

配置S2 Vlan

```
[S2]vlan 2
[S2-vlan2]port e 1/0/24
[S2-vlan2]inter vlan 2
[S2-Vlan-interface2]ip address 192.168.6.1 255.255.255.0
[S2-Vlan-interface2]quit
[S2-Vlan2]quit
[S2]vlan 1
[S2-vlan1]port e 1/0/1
[S2-vlan1]inter vlan 1
[S2-Vlan-interface1]ip address 192.168.4.2 255.255.255.0
```

步骤3 配置各设备各接口IP地址

PCA

192.168.5.2

255.255.255.0

192.168.5.1

PCB

192.168.6.2

255.255.255.0

192.168.6.1

配置R1

```
[R1]interface e 0/0
[R1-Ethernet0/0]ip address 192.168.3.1 255.255.255.0
[R1-Ethernet0/0]quit
[R1]interface s 1/0
[R1-Serial1/0]ip address 192.168.0.1 255.255.255.0
```

配置R2

```
[R2]interface e 0/0
[R2-Ethernet0/0]ip address 192.168.4.1 255.255.255.0
[R2-Ethernet0/0]quit
[R2]interface s 1/0
[R2-Serial1/0]ip address 192.168.0.2 255.255.255.0
```

步骤4 在S1 S2 R1 R2上启动ospf协议，规划area

配置S1

```
[S1]ospf
[S1-ospf]area 1
[S1-ospf-area-0.0.0.1]network 192.168.3.0 0.0.0.255
```

配置S2

```
[S2]ospf
[S2-ospf]area 2
[S2-ospf-area-0.0.0.2]network 192.168.4.0 0.0.0.255
```

配置R1

```
[R1]ospf
[R1-ospf]area 1
[R1-ospf-area-0.0.0.1]network 192.168.3.0 0.0.0.255
[R1-ospf-area-0.0.0.1]quit
[R1-ospf]area 0
[R1-ospf-area-0.0.0.0]network 192.168.0.0 0.0.0.255
```

配置R2

```
[R2]ospf
[R2-ospf]area 0
[R2-ospf-area-0.0.0.0]network 192.168.0.0 0.0.0.255
[R2-ospf-area-0.0.0.0]quit
[R2-ospf]area 2
[R2-ospf-area-0.0.0.2]network 192.168.4.0 0.0.0.255
```

步骤5 添加静态路由

S1

```
[S1]ip route-static 192.168.6.0 24 192.168.3.1
```

S2

```
[S2]ip route-static 192.168.5.0 24 192.168.4.1
```

R1

```
[R1]ip route-static 192.168.5.0 24 192.168.3.2
[R1]ip route-static 192.168.6.0 24 192.168.0.2
```

R2

```
[R2]ip route-static 192.168.5.0 24 192.168.0.1
[R2]ip route-static 192.168.6.0 24 192.168.4.2
```

步骤6 查看各个路由器交换机路由表

`display ip routing-table`

（截图）

## 8 设计实验 2

### 8.1 实验步骤

#### 8.1.1 配置

S1：
```plaintext
vlan 2
port e1/0/23
inter vlan 2
ip add 192.168.5.1 24

vlan 3
port e1/0/2
inter vlan 3
ip add 192.168.4.2 24

vlan 4
port e1/0/24
inter vlan 4
ip add 192.168.6.1 24

vlan 1
port e1/0/1
inter vlan 1
ip add 192.168.3.2 24
quit

router id 2.2.2.2

ospf 
area 0
network 192.168.3.0 0.0.0.255
network 192.168.4.0 0.0.0.255
network 192.168.5.0 0.0.0.255（存疑）
network 192.168.6.0 0.0.0.255（存疑）
inter vlan 1
ospf cost 100
inter vlan 3
ospf cost 200
quit
```

S2：
```plaintext
inter loop1
ip add 211.100.2.1 24
vlan 2
port e1/0/0
inter vlan 2
ip add 202.112.1.2 24

vlan 3
port e1/0/1
inter vlan 3
ip add 202.112.2.2 24

ip route-static 0.0.0.0 0 202.112.1.1 preference 60(存疑，可能不需要preference 60?)
ip route-static 0.0.0.0 0 202.112.2.1 preference 80(存疑)
```

R1：
```plaintext
inter e0/0
ip add 202.112.1.1 24
inter e0/1
ip add 192.168.3.1 24
inter s1/0
ip add 192.168.0.1 24
quit

router id 1.1.1.1

ospf 
area 0
network 192.168.3.0 0.0.0.255
network 192.168.0.0 0.0.0.255
inter e0/1
ospf cost 100
inter s1/0
ospf cost 200

ip route-static 211.100.2.0 24 202.112.1.2
import-route static
```

R2：
```plaintext
inter e0/0
ip add 202.112.2.1 24
inter e0/1
ip add 192.168.4.1 24
inter s1/0
ip add 192.168.0.2 24
quit

router id 3.3.3.3

ospf 
area 0
network 192.168.4.0 0.0.0.255
network 192.168.0.0 0.0.0.255

inter e0/1
ospf cost 200
inter s1/0
ospf cost 200

ip route-static 211.100.2.0 24 202.112.2.2
import-route static
```

PC1/PC2 配置
PC1：IP=192.168.5.2/24，子网掩码 = 255.255.255.0，网关 = 192.168.5.1（S1-VLAN2 接口）
PC2：IP=192.168.6.2/24，子网掩码 = 255.255.255.0，网关 = 192.168.6.1（S1-VLAN4 接口）

#### 8.1.2 验证
##### 场景 1：正常状态（所有链路畅通，主路径 S1-R1-Internet 生效）

###### 验证步骤 1：检查 OSPF 邻居（R1 上执行）

- 操作：`display ospf peer`
- 预期输出：
```plaintext
OSPF Process 1 with Router ID 1.1.1.1
Neighbors in Area 0 (2 neighbors)
Router ID: 2.2.2.2 (S1)       Address: 192.168.3.2      State: Full  Mode: N/A
Router ID: 3.3.3.3 (R2)       Address: 192.168.0.2      State: Full  Mode: N/A
```
说明：R1 与 S1、R2 的 OSPF 邻居均为 Full（正常互联）。

###### 验证步骤 2：检查 S1 的 OSPF 路由表（S1 上执行）

- 操作：`display ospf routing`
- 预期输出（核心条目）：
```plaintext
OSPF Process 1 Router ID: 2.2.2.2
Routing Table for Area 0
Destination        Cost  Type       NextHop         Interface
211.100.2.0/24     110   Type 5     192.168.3.1     Vlan-interface1
192.168.0.0/24     300   Type 2     192.168.3.1     Vlan-interface1
202.112.1.0/24     110   Type 2     192.168.3.1     Vlan-interface1
```
说明：访问 211.100.2.0/24 的下一跳是 192.168.3.1（R1），Cost=110（100（S1-R1）+10（R1-S2，默认 Cost）），主路径生效。

###### 验证步骤 3：PC1 ping + 追踪路由（PC1 上执行）

- 操作 1：`ping 211.100.2.1`
    预期输出：`Reply from 211.100.2.1: bytes=32 time<1ms TTL=62`（通）
- 操作 2：`tracert 211.100.2.1`（Windows）/ `traceroute 211.100.2.1`（Linux）
    预期输出：
```plaintext
1    <1 ms    <1 ms    <1 ms  192.168.5.1 (S1)
2    <1 ms    <1 ms    <1 ms  192.168.3.1 (R1)
3    <1 ms    <1 ms    <1 ms  202.112.1.2 (S2)
4    <1 ms    <1 ms    <1 ms  211.100.2.1 (S2环回口)
```
说明：流量走主路径 S1→R1→S2，符合题目 “指定路径为 S1-R1-Internet” 要求。

##### 场景 2：S1-R1 链路故障（断开 S1-e1/0/1 或 R1-e0/1）

###### 验证步骤 1：检查 S1 的 OSPF 路由表（S1 上执行）

- 操作：`display ospf routing`
- 预期输出（核心条目）：
```plaintext
OSPF Process 1 Router ID: 2.2.2.2
Routing Table for Area 0
Destination        Cost  Type       NextHop         Interface
211.100.2.0/24     210   Type 5     192.168.4.1     Vlan-interface3
192.168.0.0/24     400   Type 2     192.168.4.1     Vlan-interface3
202.112.2.0/24     210   Type 2     192.168.4.1     Vlan-interface3
```
说明：下一跳切换为 192.168.4.1（R2），Cost=210（200（S1-R2）+10（R2-S2，默认 Cost）），备份路径 1 生效。

###### 验证步骤 2：PC1 ping + 追踪路由（PC1 上执行）

- 操作 1：`ping 211.100.2.1`
    预期输出：`Reply from 211.100.2.1: bytes=32 time<1ms TTL=61`（通）
- 操作 2：`tracert 211.100.2.1`
    预期输出：
```plaintext
1    <1 ms    <1 ms    <1 ms  192.168.5.1 (S1)
2    <1 ms    <1 ms    <1 ms  192.168.4.1 (R2)
3    <1 ms    <1 ms    <1 ms  202.112.2.2 (S2)
4    <1 ms    <1 ms    <1 ms  211.100.2.1 (S2环回口)
```
说明：流量自动切换到 S1→R2→S2，符合题目 “故障时选 S1-R2-Internet” 要求。

##### 场景 3：S1-R1 + R2-S2 链路双故障（断开 S1-e1/0/1 + R2-e0/0）

###### 验证步骤 1：检查 S1 的 OSPF 路由表（S1 上执行）

- 操作：`display ospf routing`
- 预期输出（核心条目）：
```plaintext
OSPF Process 1 Router ID: 2.2.2.2
Routing Table for Area 0
Destination        Cost  Type       NextHop         Interface
211.100.2.0/24     410   Type 5     192.168.4.1     Vlan-interface3
192.168.0.0/24     400   Type 2     192.168.4.1     Vlan-interface3
202.112.1.0/24     410   Type 2     192.168.4.1     Vlan-interface3
```
说明：下一跳仍为 192.168.4.1（R2），Cost=410（200（S1-R2）+200（R2-R1）+10（R1-S2）），备份路径 2 生效。

###### 验证步骤 2：PC1 ping + 追踪路由（PC1 上执行）

- 操作 1：`ping 211.100.2.1`
    预期输出：`Reply from 211.100.2.1: bytes=32 time=1ms TTL=60`（通）
- 操作 2：`tracert 211.100.2.1`
    预期输出：
```plaintext
1    <1 ms    <1 ms    <1 ms  192.168.5.1 (S1)
2    <1 ms    <1 ms    <1 ms  192.168.4.1 (R2)
3    <1 ms    <1 ms    <1 ms  192.168.0.1 (R1)
4    <1 ms    <1 ms    <1 ms  202.112.1.2 (S2)
5    <1 ms    <1 ms    <1 ms  211.100.2.1 (S2环回口)
```
说明：流量自动切换到 S1→R2→R1→S2，符合题目 “R2-Internet 故障时选 S1-R2-R1-Internet” 要求。

# 9 讨论区
1. 关于线下上机的 tips
    1. 依旧用自己电脑，依旧虚拟机。
    2. 和线上上机的区别只有线下+要当场提交一份实验报告（没写完的可以课下接着补，大部分人都写不完所以不用担心）
    3. 机房的砖凹凸不平，或者说每块砖的高度朝向都不一样，**小心行走**
    4. 如果你和你的队友是一人负责一个实验这种的，记得在不做实验的时候也要电脑亮屏，打开虚拟机（不过老师就算发现了也只是说几句，貌似不会有啥后果）
    5. 如果你是先写一份电子版再誊抄成纸质版，建议用手机看报告，和 4 一样，老师貌似不支持这种行为，但依旧不会有什么严重后果