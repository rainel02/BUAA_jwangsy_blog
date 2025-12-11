---
title: 【计网实验】lab3-网络层
date: 2025-11-23 21:49:30
tags: 学习经验
categories:
- [学习经验, 计网实验]
math: true
---
## 1 本次实验内容

1. ARP协议实验
2. ICMP协议实验
3. IP协议实验
4. VLAN间路由综合实验（选做）
5. 子网划分设计实验（选做）

## 2 MOOC 答案

![01](https://hackmd.io/_uploads/HyQODJ_G-g.png)
![02](https://hackmd.io/_uploads/Bk7uD1OfWe.png)
![03](https://hackmd.io/_uploads/H1mdDkOf-g.png)


## 3 ARP协议实验

实验内容：通过在位于同一网段和不同网段的主机之间执行ping命令，截获报文，分析ARP报文结构，并分析ARP在同一网段内和不同网段间的解析过程。

### 3.1 同一网段的ARP分析

步骤1 首先，进入PCA的连线组网程序，按照课本图3-3进行组网，提交组网，确保交换机配置被清空，配置IP地址PCA 192.168.1.22 255.255.255.0 PCB 192.168.1.21 255.255.255.0

步骤2 在PCA上打开命令行窗口，执行命令`arp-a`，显示<u>没有ARP的记录</u>（对应实验报告填空）（截图），在PCB里重复以上过程，显示<u>没有ARP的记录</u>（截图）；如果有记录，请执行`arp-d`清空ARP缓存。

步骤3 在PCA和PCB上打开Wireshark软件，Capture Options选择**本地连接**，开始截取报文，在**PCA**命令行窗口上执行`ping 192.168.1.21`（ping B）。执行完毕后可以停止PCA, PCB上的Wireshark报文截获，将此次报文截获结果（都保存保险点）保存文件名为"ping1-学号"（截图）

步骤4 在PCA和PCB上命令行窗口再次执行`arp-a`命令，观察结果（对应实验报告填空）（截图）

步骤5 重复步骤3。将此次结果（两台机器分别都保存）保存文件名为"ping2-学号"（截图）

步骤6 分析文件"ping1-学号"，填写信息（对应实验报告填空），观察报文数量，ARP报文内容，Opcode字段信息，将ARP请求报文与ARP应答报文字段信息填写进入实验报告中（对应实验报告填空，建议截图保留相关信息）

步骤7 分析文件"ping2-学号"，对比与步骤6分析的结果，缺少什么报文，简述缓存作用，填写实验报告相关内容（可以截图保留信息）

步骤8 按照课本图3-4进行重新组网，提交组网，配置PCA和PCB的IP地址，PCA 192.168.1.22 255.255.255.0 网关192.168.1.10 PCB 192.168.2.22 255.255.255.0 网关192.168.2.10，接着为交换机设置VLAN2和VLAN3，启动超级终端（，为超级终端任意命名，连接时假定使用COM1端口，把端口设置里的每秒位数设置为9600，数据流控制设置为无，单击确定），进入后按Enter进入相关视图。

### 3.2 不同网段的ARP分析

步骤1 设置交换机的VLAN2，命令如下：

```
<h3c>system
[h3c]sysname S1
[S1]vlan 2
[S1-vlan2]port ge 1/0/1
[S1-vlan2]inter vlan 2
[S1-Vlan-interface2]ip add 192.168.1.10 255.255.255.0
```

继续配置VLAN3

```
[S1-Vlan-interface2]vlan 3
[S1-vlan3]port ge 1/0/13
[S1-vlan3]inter vlan 3
[S1-Vlan-interface3]ip add 192.168.2.10 255.255.255.0
```

注意：完成配置后先不要用ping检查连通性，如果检查了，请执行`arp-d`清空缓存

步骤2 运行PCA和PCB的Wireshark软件，开始截获数据报文；在PCA上执行`ping 192.168.2.22`命令，执行后停止PCA和PCB的截获，将结果保存为"ping3-学号"（可以截图）

步骤3 在PCA命令行执行`arp-a`观察结果（对应实验报告填空）（截图）

步骤4 分析截获报文内容，选中第一条ARP请求报文和第一条ARP应答报文，字段信息与之前填写的表进行对比，与相同网段比较，有什么异同（对应实验报告简答题）

## 4 ICMP 协议实验

### 4.1 实验步骤

#### 4.1.1 ICMP 询问报文分析

1. 组网
	![04](https://hackmd.io/_uploads/ByVFPkdzWg.png)

	1. PCA Eth0 与 S1 的 E0/1 连
	2. PCB Eth0 与 S1 的 E0/23 连
	3. 实验组网-提交
2. 为 PCA、PCB 配置 ip 地址和默认网关，为交换机配置 VLAN2 和 VLAN3
	1. PCA
		IP 地址：10.1.2.10
		子网掩码：255.255.255.0
		默认网关：10.1.2.1
	2. PCB
		IP 地址：10.1.3.10
		子网掩码：255.255.255.0
		默认网关：10.1.3.1
	3. 配置交换机
		1. （我觉得直接打开 s1.ht 也可以）启动超级终端，取个名字 test，然后设置端口
			![05](https://hackmd.io/_uploads/Hym5vJuz-x.png)

		2. 配置交换机
			`sys`
			`sysname S1`
			`vlan 2`
			`port e 0/1` e 不行就 ge 或者 gigabitEthernet
			`inter vlan2`
			`ip add 10.1.2.1 255.255.255.0`
			`vlan 3`
			`port e 0/23` e 不行就 ge 或者 gigabitEthernet
			`inter vlan3`
			`ip add 10.1.3.1 255.255.255.0`
3. PCA、PCB 上启动 wireshark 进行报文截获
4. PCA ping PCB `ping 10.1.3.10`
	***完成实验报告第 6 题***
5. 重新用 wireshark 进行报文截获，并 PCA 上启动 pingtest ![06](https://hackmd.io/_uploads/HkRcwJuG-l.png)

	注：如果pingtest软件出现“发送ICMP数据报文出现错误”，就以管理员身份运行PingTest软件
	1. 设置地址掩码请求报文参数
		![07](https://hackmd.io/_uploads/ry_owyOM-x.png)

	2. 点击“发送”
	3. 查看 wireshark 截获的 ICMP 报文
		***完成实验报告第 7 题***
1. PCA 上重新启动 pingtest
	1. 设置时间戳请求报文参数
		![08](https://hackmd.io/_uploads/Hyz3D1OGbe.png)

	2. 点击“发送”
	3. 查看 wireshark 截获的 ICMP 报文
		***完成实验报告第 8 题***

#### 4.1.2 ICMP 差错报文分析

1. 重新用 wireshark 进行报文截获
2. PCA 上 `ping 10.1.3.20`
3. PCA 上 `ping 10.1.4.10`
4. 分析上面两种情况产生的报文。
   （注：有可能两种情况都显示timed out，参考[计网实验常见问题](https://docs.qq.com/doc/DVXJTdkV1UFJFcWZz)的第47条）
	在超级终端里
	sys
	ip ttl-expires enable
	ip unreachables enable
	***完成实验报告第 9 题***

5. 取消 PCA 上的 DNS 配置（确定 DNS 服务器都为空）
	![09](https://hackmd.io/_uploads/SJJ6wJuGWl.png)

6. PCA, PCB重新用 wireshark 进行报文截获
7. PCA 上命令行 `tracert 10.1.3.10`
8. 保存 PCA 截获的报文为"tracert-23373125"并进行分析
	***完成实验报告第 10 题***

### 4.2 报告填写

#### 6 根据3.6.1步骤2——在PC A 和 PC B上启动Wireshark软件进行报文截获，然后PC A ping PC B，分析截获的ICMP报文： 共有个ICMP报文，分别属于哪些种类？对应的种类和代码字段分别是什么？请分析报文中的哪些字段保证了回送请求报文和回送应答报文的一一对应？

1. 8 个 ICMP 报文，4 个 request，4个reply
2. ![12](https://hackmd.io/_uploads/HyjpPkuMbx.png)

3. 标识符 Identifier 和序列号 Sequence number字段保证了回送请求报文和回送应答报文的一一对应。

#### 7 根据3.6.1步骤3——在 PCA 和 PCB 上启动Wireshark软件进行报文截获，运行pingtest程序，设置地址掩码请求报文参数，分析截获报文填写下表：

|                      |                           |                      |                            |
| -------------------- | ------------------------- | -------------------- | -------------------------- |
| 地址掩码请求报文             |                           | 地址掩码应答报文             |                            |
| ICMP字段名              | 字段值                       | ICMP字段名              | 字段值                        |
| Type                 | 17 (Address mask request) | Type                 | 18 (Address mask reply)    |
| Code                 | 0                         | Code                 | 0                          |
| Checksum             | Oxe3ff[correct]           | Checksum             | Oxe3fe [correct]           |
| Identifier(BE)       | 2560 (0x0a00)             | Identifier (BE)      | 2560 (0x0a00)              |
| Identifier(LE)       | 10 (0x000a)               | Identifier (LE)      | 10 (0x000a)                |
| sequence number(BE)  | 256 (0X0100)              | sequence number (BE) | 256 (0X0100)               |
| sequence number (LE) | 1 (0X0001)                | sequence number (LE) | 1 (0x0001)                 |
| Address mask         | 0.0.0.0 (0X00000000)      | Address mask         | 255.255.255.0 (0xffffffQ0) |

#### 8 根据3.6.1步骤4——在 PCA 和 PCB 上启动Wireshark软件进行报文截获，运行pingtest程序，设置时间戳请求报文参数，分析截获报文填写下表：

![13](https://hackmd.io/_uploads/H1ERPJdzWe.png)


通过上述实验，仔细体会ICMP询问报文的作用。

#### 9 根据3.6.2中步骤1回答：

1. 请比较这两种情况有何不同？
	对于10.1.3.20，处于不同的网段，发送至网关后能够找到相应的网络，但在网络内发送ARP请求不能获得应答，无法继续转发，因而不能获得应答；而对于10.1.4.10，在发送到网关，也就是S1后，由于S1路由表中并没有该网络的项，认为不可达，并向源主机发送重点不可达报文。
2. 截获了哪种ICMP差错报文？其类型和代码字段值是什么？此报文的ICMP协议部分又分为了几部分？其作用是什么？
	截获到了终点不可达报文，类型为3（Destination unreachable），代码为0（Network unreachable），表示网络不可达。此报文的ICMP协议部分分为两部分，第一部分为前四个字节，包括类型、代码和校验，是ICMP的统一部分；对于此类ICMP报文，剩下的是数据部分，即产生错误的IP数据报的20字节首部和其后8字节的数据，用来表示产生错误的原始ICMP报文。

#### 10 根据3.6.2中步骤2回答：

1. 结合报文内容，简述tracert的工作过程。
	分析内容可知：
	1. 第一步，发送一个 TTL 为 1 的报文，当该报文到达第一个路由器时 TTL 减 1 变为 0，路由器就不再对其进行转发，直接丢弃。此时发送一个ICMP“超时”信息给源主机，并告知自己的IP地址。
	2. 第二步，再发送一个TTL为2的报文，与以上的过程相似，在第二跳返回TTL超时，这个过程不断进行，直至到达目的地。
	3. 在目的地，因为此时数据报使用的是无效的端口号（默认为33434），目的主机会返回一个ICMP目的地不可达消息，从而tracert操作结束。
	在以上的数据传输过程中，tracert会记录下每一个ICMP TTL超时消息的源地址，从中获得报文到达目的地所经过的网关的IP地址。
2. 截获了哪种ICMP差错报文？其类型和代码字段值是什么？
	截获到了ICMP超时差错报文，类型（Type）:11，代码字段（Code）：0。

## 5 IP协议实验

实验内容：结合上一个实验的报文，分析IP报文格式，然后结合实验体会IP地址的编址方法和数据报文发送、转发过程；最后分析路由表的结构和功能。

步骤1 组网与课本图3-10相同（上一实验保留）。在上一实验3.6.2步骤2基础上，用Wireshark软件打开文件"tracert-学号"，分析IP报文（考虑截图保留证据）。写出tracert指令用到了IP协议报文的哪几个字段（对应实验报告简答题）

步骤2 将PCA子网掩码配置为255.255.0.0，在PCA和PCB上启动Wireshark软件进行报文截获，然后PCApingPCB`ping 10.1.3.10`，观察PCA和PCB能否ping通（考虑截图保留信息），结合截获报文分析原因（对应实验报告简答）

步骤3 将PCA子网掩码恢复为255.255.255.0。查看交换机路由信息，在交换机超级终端中执行命令

```
[H3C]display ip routing-table
```

将得到的结果截图保留（对应实验报告填表）

步骤4* 取消交换机的三层转发功能（考虑截图保留操作证据）

```
[S1]inter vlan 2
[S1-Vlan-interface2]undo ip add 10.1.2.1 255.255.255.0
[S1-Vlan-interface2]inter vlan 3
[S1-Vlan-interface3]undo ip add 10.1.3.1 255.255.255.0
```

步骤5* 执行PCApingPCB`ping 10.1.3.10`（考虑截图保留操作过程），查看路由表信息`display ip routing-table`（截图保留操作证据），比较结果，体会作用

步骤6 按照实验2课本图2-17连接组网，提交组网，配置路由器，两个路由器互相ping，观察能否ping通（考虑截图保留证据），执行以下命令（保留结果信息）

```
<R1>debugging ppp all
<R1>terminal debugging
[R2-Serial1/0]shutdown
[R2-Serial1/0]undo shutdown
<R1>undo terminal debugging
```

步骤7 将路由器R2的接口S1/0的IP地址改为10.0.0.1/24

```
[R2-Serial1/0]undo ip address 192.0.0.2 24
[R2-Serial1/0]ip address 10.0.0.1 24
```

再看两台路由器能否ping通（截图） （对应实验报告简答题）

## 6 网络层分片实验（看样子不要做）

实验内容：通过在路由器与计算机之间传送数据报文，设置MTU的大小，将报文分片，验证分片过程

步骤1 按照课本图3-15连接组网，配置计算机IP地址，确保清空路由器和交换机配置信息。

步骤2 配置路由器，设置路由器以太网端口MTU为100B，如下：

```
[H3C]inter GigabitEthernet 0/0(看情况而定)
[H3C-GigabitEthernet0/0]mtu 100
[H3C-GigabitEthernet0/0]ip add 192.192.169.10 255.255.255.0
```

步骤3 打开PCA, PCB, PCC, PCD上的Wireshark软件，开始报文截获

步骤4 将ping命令的数据包数据部分大小设置为300字节，这样数据包只能分片。执行以下命令：

```
[H3C-GigabitEthernet0/0]ping -s 300 192.192.169.21 // 向A发送
[H3C-GigabitEthernet0/0]ping -s 300 192.192.169.22 // 向B发送
[H3C-GigabitEthernet0/0]ping -s 300 192.192.169.23 // 向C发送
[H3C-GigabitEthernet0/0]ping -s 300 192.192.169.24 // 向D发送
```

步骤5 各自分析在计算机上截获的数据报文

......略

## 7 VLAN 间路由综合实验（选做）

### 7.1 实验步骤

1. 组网
	![10](https://hackmd.io/_uploads/BybJuy_f-g.png)
	![11](https://hackmd.io/_uploads/Sk31O1dG-x.png)

2. 配置 IP 地址（网关暂时先不配置）
	1. PCA
		IP 地址：192.168.2.10
		子网掩码：255.255.255.0
	2. PCB
		IP 地址：192.168.3.10
		子网掩码：255.255.255.0
	3. PCC
		IP 地址：192.168.2.11
		子网掩码：255.255.255.0
	4. PCD
		IP 地址：192.168.3.11
		子网掩码：255.255.255.0
3. 配置 s1 和 s2
	1. 配置 s1：PCA 上 s1.ht 超级终端内
		`sys`
		`sysname S1`
		`vlan 2`
		`port e 0/1 to e 0/5` e 不行就 ge 或者 gigabitEthernet
		`vlan 3`
		`port e 0/20 to e 0/24` e 不行就 ge 或者 gigabitEthernet
	2. 配置 s2：PCB 上 s2.ht 超级终端内
		`sys`
		`sysname S2`
		`vlan 2`
		`port e 0/1 to e 0/5` e 不行就 ge 或者 gigabitEthernet
		`vlan 3`
		`port e 0/20 to e 0/24` e 不行就 ge 或者 gigabitEthernet `
4. 配置交换机上的 trunk 端口：
	1. PCA 上 s1.ht 超级终端内
		`inter e 0/13`
		`port link-type trunk`
		`port trunk permit vlan 2 3`
	2. PCB 上 s2.ht 超级终端内
		`inter e 0/13`
		`port link-type trunk`
		`port trunk permit vlan 2 3`
5. 四台计算机上都运行 wireshark 截获报文
	注：为了保证PCB截获带VLAN标签的报文，需要在“控制面板--网络连接”界面，将PCB的本地连接接口重启（先禁用，再启用）。然后再启动wireshark软件截获报文，这样就可以截获带VLAN标签的报文。
6. PCC ping PCD `ping 192.168.3.11` 发现不能 ping 通
	根据各个计算机截获的报文分析原因

7. 在交换机S1上配置VLAN2和VLAN3的接口P地址,VLAN2接口的IP地址配置为192.168.2.1/24.VLAN3接口的IP地址配置为192.168.3.1/24
	`inter VLAN 2`
	`ip address 192.168.2.1 255.255.255.0`
	`inter VLAN 3`
	`ip address 192.168.3.1 255.255.255.0`
8. 配置各台计算机的默认网关地址
	PCA：192.168.2.1
	PCB:192.168.3.1
	PCC:192.168.2.1
	PCD:192.168.3.1
9.  清空所有计算机和交换机的MAC地址表和arp缓存
	1. PCA,PCB,PCC,PCD 上 cmd
		`arp -d`（注：如果如果 `arp -d` 命令提示“ARP项删除失败：请求的操作需要提升”，就以管理员身份运行CMD）
		`arp -a`（查看 arp 缓存发现全部清空了）
	2. PCA,PCB 上 s1.ht\/s3.ht 超级终端
		`quit` （到\[s1\]）
		`undo mac-address` (y)
		`reset arp all`
10. PCC ping PCD `ping 192.168.3.11` 发现能 ping 通
	根据各个计算机截获的报文分析，***完成实验报告第 16 题***

### 7.2 报告填写（以下答案存疑，建议跟着实际报文写）

16．综合型实验（VLAN间路由实验结果分析）（选做）
根据跨交换机VLAN间路由实验（PCC ping PCD）所截获报文，对整个网络层和数据链路层的报文转发过程进行分析。

**约定如下：数据帧中的MAC地址对：（目的MAC地址，源MAC地址）**

**数据报中的IP地址对：（目的IP地址，源IP地址）**

##### **STEP 1**

1. PCC发送的第一个报文类型是什么？为什么？    
	是ARP报文，因为发送要首先获得目的主机的MAC地址。
2. 包含该报文数据帧中的VLAN id、MAC和IP地址对是：
    1. VLAN id＝2
    2. MAC：(ff.ff.ff.ff.ff.ff, MAC_PCC)
    3. IP：([192.168.2.1](192.168.2.1), [192.168.2.11](192.168.2.11))

##### **STEP 2**

1. S2收到数据帧后，对其MAC地址表的操作是：
	在其MAC地址表增加一条记录，内容为（MAC_PCC，E0/1）。
2. S2根据接收数据帧的端口所属VLAN，在其中插入VLAN id＝2的标签，并向除接收端口外的所有VLAN2端口转发这个数据帧。

##### **STEP 3**

1. S1收到数据帧后，对其MAC地址表的操作是：
	在其MAC地址表增加一条记录，内容为（MAC_PCC，E0/13）。
2. S1将ARP报文交付给网络层，对其ARP表的操作是：
	在其ARP缓存增加一条记录，内容为(MAC_PCC, [192.168.2.11](192.168.2.11))。
3. S1发送的包含ARP Reply报文的数据帧中：
    1. MAC：(MAC_PCC，MAC_VLAN2)
    2. IP：([192.168.2.11](192.168.2.11), [192.168.2.1](192.168.2.1))
    3. VLAN id＝2

##### **STEP 4**

1. S2收到数据帧后，对其MAC地址表的操作是：
	在其MAC地址表增加一条记录，内容为（MAC_VLAN2，E0/13）。
2. S2收到数据帧后，根据VLAN标签和MAC表，决定向端口E0/1转发该数据帧。
3. S2根据端口E0/1是access类型端口，去掉VLAN标签，从端口E0/1转发该帧。

##### **STEP 5**

1. PCC收到ARP Reply报文，更新其ARP缓存。显示ARP缓存的命令：`arp -a`，显示的内容：
    1. IP: [192.168.2.1](192.168.2.1)
    2. MAC: MAC_VLAN2
2. PCC发送的包含ICMP Echo Request报文的数据帧中：
    1. VLAN id＝2
    2. MAC：(MAC_VLAN2，MAC_PCC)
    3. IP：([192.168.3.11](192.168.3.11)，[192.168.2.11](192.168.2.11))

##### **STEP 6**

1. S2收到数据帧，根据其接收端口添加VLAN2标签；根据目的MAC查找MAC地址表；将数据帧由E0/13端口转发给S1。
2. S2转发的数据帧中：
    1. VLAN id＝2
    2. MAC：(MAC_VLAN2，MAC_PCC)
    3. IP：([192.168.3.11](192.168.3.11)，[192.168.2.11](192.168.2.11))

##### **STEP 7**

1. S1收到S2转发的数据帧，交付网络层，根据目的IP地址查路由表，将报文路由到int vlan 3，准备通过数据链路层交付给PCD。
2. 因未查到PCD的MAC地址，需发送包含ARP Request报文的数据帧：
    1. VLAN id＝3
    2. MAC：(ff.ff.ff.ff.ff.ff，MAC_VLAN3)
    3. IP：([192.168.3.11](192.168.3.11)，[192.168.3.1](192.168.3.1))

##### **STEP 8**

1. S2收到S1转发的数据帧，根据其VLAN id＝3，向除接收端口外的所有属于VLAN3的端口转发该数据帧。
2. S2根据端口E0/24是access类型端口，去掉VLAN标签，从端口E0/24转发该帧。

##### **STEP 9**

1. PCD收到S2转发的数据帧，更新其ARP缓存，内容为：
    1. IP: [192.168.3.1](192.168.3.1)
    2. MAC: MAC_VLAN3
2. PCD发送包含ARP Reply报文的数据帧中：
    1. VLAN id＝3
    2. MAC：(MAC_VLAN3，MAC_PCD)
    3. IP：([192.168.3.1](192.168.3.1)，[192.168.3.11](192.168.3.11))

##### **STEP 10**

1. S2收到数据帧，根据其接收端口添加VLAN3的标签；根据目的MAC查找MAC地址表；将数据帧由E0/13端口转发给S1。
2. S2转发的数据帧中：
    1. VLAN id＝3
    2. MAC：(MAC_VLAN3，MAC_PCD)
    3. IP：([192.168.3.1](192.168.3.1)，[192.168.3.11](192.168.3.11))

##### **STEP 11**

1. S1收到数据帧，提交到网络层，更新其ARP表。
2. S1对包含ICMP Echo Request报文的数据帧的VLAN标签进行替换，由VLAN id=2变为VLAN id=3。封装的数据帧中：
    1. VLAN id＝3
    2. MAC：(MAC_PCD，MAC_VLAN3)
    3. IP：([192.168.3.11](192.168.3.11)，[192.168.3.1](192.168.3.1))
3. 查找MAC地址表，由E0/13端口发送。

##### **STEP 12**

1. S2收到S1转发的数据帧，根据其VLAN id和目的MAC地址，向E0/24端口转发该数据帧。
2. S2根据端口E0/24是access类型端口，去掉VLAN标签，从端口E0/24转发该帧。

##### **STEP 13**

1. PCD收到包含ICMP Echo Request报文的数据帧，发送包含ICMP Echo Reply报文的数据帧：
    1. VLAN id＝3
    2. MAC：(MAC_VLAN3，MAC_PCD)
    3. IP：([192.168.3.1](192.168.3.1)，[192.168.3.11](192.168.3.11))

##### **STEP 14**

1. S2收到数据帧，根据其接收端口添加VLAN3的标签；根据目的MAC查找MAC地址表；将数据帧由E0/13端口转发给S1。
2. S2转发的数据帧中：
    1. VLAN id＝3
    2. MAC：(MAC_VLAN3，MAC_PCD)
    3. IP：([192.168.3.1](192.168.3.1)，[192.168.3.11](192.168.3.11))

##### **STEP 15**

1. S1收到S2转发的数据帧，交付网络层，根据目的IP地址查路由表，将报文路由到int vlan2，准备通过数据链路层交付给PCC。
2. 查找PCC的MAC地址，替换VLAN标签，封装并发送数据帧：
    1. VLAN id＝2（原内容空缺，根据上下文补充）
    2. MAC：(MAC_VLAN3，MAC_PCD)
    3. IP：([192.168.3.1](192.168.3.1)，[192.168.3.11](192.168.3.11))

##### **STEP 16**

1. S2收到S1转发的数据帧，根据其VLAN id和目的MAC地址，向E0/1端口转发该数据帧。
2. S2根据端口E0/1是access类型端口，去掉VLAN标签，从端口E0/1转发该帧。

这样，PCC收到S2转发的包含ICMP Echo Reply报文的数据帧，第一轮ICMP询问和应答过程结束。

## 8 子网划分设计实验（选做）

**注意！！！在配置前务必清空之前的配置，否则以下配置可能无效**
**注意！！！在配置前务必清空之前的配置，否则以下配置可能无效**
**注意！！！在配置前务必清空之前的配置，否则以下配置可能无效**

根据题意，要求每个子网的主机数大于15，由于

$2^4 - 2 = 14$

$2^5 - 2 = 30$

主机号至少需要5位，故可留3位用于划分子网

![15(1)](https://hackmd.io/_uploads/ry2XOkdf-g.png)


如图划分

接下来在虚拟机上进行配置

步骤1 连线组网，选用一个路由器R1，两个交换机S1，四台电脑，按照课本的示意图先配好已有信息

```
R1 E0/0 - S1 E0/1
S2 E0/13 - S1 E0/13
PCA E0 - S2 E0/1
PCB E0 - S2 E0/24
PCC E0 - S1 E0/23
PCD E0 - S1 E0/24
```

步骤2 配置IP地址A-D

202.108.100.1

255.255.255.224

202.108.100.2



202.108.100.30

255.255.255.224

202.108.100.2



202.108.100.225

255.255.255.224

202.108.100.226



202.108.100.254

255.255.255.224

202.108.100.226

步骤3 配置交换机和路由器相关信息

配置R1

```
<H3C>sy
[H3C]sysname R1
[R1]ip route-static 202.108.100.2 255.255.255.224 211.100.217.193
[R1]ip route-static 202.108.100.226 255.255.255.224 211.100.217.193
[R1]interface ge 0/0
[R1-GigabitEthernet0/0]ip add 211.100.217.192 24
```

配置S1

```
<H3C>sy
[H3C]sysname S1
[S1]vlan 2
[S1-vlan2]port ge 1/0/13
[S1-vlan2]inter vlan2
[S1-Vlan-interface2]ip add 202.108.100.2 255.255.255.224
[S1-Vlan-interface2]vlan 3
[S1-vlan3]port ge 1/0/23 ge 1/0/24
[S1-vlan3]inter vlan3
[S1-Vlan-interface3]ip add 202.108.100.226 255.255.255.224
[S1-Vlan-interface3]vlan 1
[S1-vlan1]port ge 1/0/1 to ge 1/0/12 ge 1/0/14 to ge 1/0/22 ge 1/0/25 to ge 1/0/28(看情况定)
[S1-vlan1]inter vlan1
[S1-Vlan-interface1]ip add 211.100.217.193 24
```

联通测试(四台电脑上)

```
ping 211.100.217.192
```

