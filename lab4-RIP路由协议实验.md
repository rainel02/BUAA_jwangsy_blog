---
title: 【计网实验】lab4-RIP路由协议实验
date: 2025-11-30 17:59:30
tags: 学习经验
categories:
- [学习经验, 计网实验]
math: true
---

这次的实验基本上都没有操作录屏，极坏的……

## 1 本次实验内容

1. 静态路由及默认路由配置 - by Tengpaz
2. RIP 配置及 RIPv1 报文分析实验 - by rainel
3. 距离矢量算法实验 - by Tengpaz
4. 触发更新和水平分割实验 - by rainel
5. RIPv2 报文结构分析实验 - by Tengpaz
6. 设计型实验 （只做设计实验 2） - by rainel

## 2 MOOC 答案

![01](https://hackmd.io/_uploads/H1okI1OMWe.png)
![02](https://hackmd.io/_uploads/Bki1UJuM-e.png)
![03](https://hackmd.io/_uploads/Hyiy81uz-g.png)
![04](https://hackmd.io/_uploads/r1oy8J_zZx.png)

## 3 静态路由及默认路由配置

实验内容：在路由器/三层交换机上依次配置静态路由、默认路由，然后分别用ping命令测试网络的连通性。

### 实验步骤

步骤1 按课本图6-2所示的组网图连接好设备，配置各路由器的各接口的IP地址，及各台PC的IP地址、子网掩码和默认网关等。

配置交换机S1（在PCA超级终端上操作）

```
<H3C>system-view
[H3C]sysname S1
[S1]vlan 2
[S1-vlan2]port ge 1/0/17 to ge 1/0/24
[S1-vlan2]quit
[S1]interface vlan 1
[S1-Vlan-interface1]ip addr 192.168.2.1 255.255.255.0
[S1-Vlan-interface1]interface vlan 2
[S1-Vlan-interface2]ip addr 192.168.1.1 255.255.255.0
```

配置路由器R1（在PCC超级终端上操作）

```
<Router>system
[Router]sysname R1
[R1]interface ge 0/0
[R1-GigabitEthernet0/0]ip addr 192.168.1.2 255.255.255.0
```

配置其余四台电脑IP地址

PCA

192.168.2.2

255.255.255.0

192.168.2.1

PCB

192.168.2.3

255.255.255.0

192.168.2.1

PCC

192.168.2.4

255.255.255.0

192.168.2.1

PCD

192.168.2.5

255.255.255.0

192.168.2.1

此时4台计算机和S1之间，R1和S1之间都可以互相通信。

查看R1的路由表

```
[R1]display ip routing-table
```

（考虑截图保留操作过程）

在R1上ping其余4台计算机（考虑截图）

（对应实验报告T1简答）

步骤2 在R1上配置一条到192.168.2.0/24的静态路由。

```
[R1]ip route-static 192.168.2.0 255.255.255.0 192.168.1.1
```

观察R1路由表

```
[R1]display ip routing-table
```

（考虑截图保留操作过程）

此时路由表应该会多一条静态项

在R1上ping其余4台计算机（考虑截图）

（对应实验报告T2）

步骤3 删除刚才配置的静态路由，在R1上配置一条默认路由。默认路由也是一种静态路由，其目的地址和掩码都是0.0.0.0，该路由表项可与任何地址匹配。

```
[R1]undo ip route-static 192.168.2.0 255.255.255.0
[R1]ip route-static 0.0.0.0 0.0.0.0 192.168.1.1
```

观察R1路由表

```
[R1]display ip routing-table
```

（截图）

在R1上ping其余4台计算机（考虑截图）

（对应实验报告T3）

### 实验总结

（对应实验报告T5、T6）

| 静态路由 | ip route-static 192.168.2.0 255.255.255.0 192.168.1.1 |
| -------- | ----------------------------------------------------- |
| 缺省路由 | ip route-static 0.0.0.0 0.0.0.0 192.168.1.1           |
| RIP协议  | network 192.168.1.0                                   |


## 4 RIP 配置及 RIPv1 报文分析实验

### 4.1 实验步骤

##### 1 实验组网

![05](https://hackmd.io/_uploads/rkxWI1dfWx.png)


1. 配置 PCA PCB PCC PCD

| 设备  | IP 地址          | 子网掩码          | 网关地址        |
| --- | -------------- | ------------- | ----------- |
| PCA | 192.168.2.2/24 | 255.255.255.0 | 192.168.2.1 |
| PCB | 192.168.2.3/24 | 255.255.255.0 | 192.168.2.1 |
| PCC | 192.168.2.4/24 | 255.255.255.0 | 192.168.2.1 |
| PCD | 192.168.2.5/24 | 255.255.255.0 | 192.168.2.1 |

2. 配置 S1

```plaintext
# PCA s1.ht
<H3C>sys
[H3C]sysname S1

# vlan1 创建VLAN并划分端口 + 配置VLAN接口IP
[S1]vlan 1
[S1-vlan1]port ge 1/0/1 to 1/0/16
[S1-vlan1]inter vlan1
[S1-Vlan-interface1]ip address 192.168.2.1 255.255.255.0
[S1-Vlan-interface1]quit

# vlan2 创建VLAN并划分端口 + 配置VLAN接口IP
[S1]vlan 2
[S1-vlan2]port ge 1/0/17 to 1/0/24
[S1-vlan2]inter vlan2
[S1-Vlan-interface2]ip address 192.168.1.1 255.255.255.0
[S1-Vlan-interface2]quit
```

3. 配置 R1

```plaintext
# PCC r1.ht
<H3C>sys
[H3C]sysname R1

# 1. 配置接口IP
[R1]inter ge 0/0
[R1-Ethernet0/0]ip add 192.168.1.2 255.255.255.0
[R1-Ethernet0/0]quit
```

##### 2 在各台计算机上运行 Wireshark，然后再 S1 和 R1 上分别配置 RIP

```plaintext
# 取消默认路由并在网段192.168.1.0上启动RIP协议
[R1]undo ip route-static 0.0.0.0 0.0.0.0
[R1]rip
[R1-rip-1]network 192.168.1.0

# 在S1上为两个网段启动RIP协议
[S1]rip
[S1-rip-1]network 192.168.1.0
[S1-rip-1]network 192.168.2.0

# 配置完成后查看路由器的路由表，可以看到192.168.2.0/24的表项的协议是RIP，Pre和Cost的值也发生了变化。
[R1-rip-1]display ip routing-table

# R1上ping各台主机，发现能够ping通，直接原因是R1的路由表中有了该网络的相关表项，而该表项来自于RIP协议。
<R1>ping 192.168.2.2
<R1>ping 192.168.2.3
<R1>ping 192.168.2.4
<R1>ping 192.168.2.5
```

##### 3 分析 RIP 响应报文

![06](https://hackmd.io/_uploads/SyoWUkOfWg.png)


### 4.2 实验报告

4. 在配置RIP协议后，比较和步骤1中R1路由表的差异；测试R1和各台计算机是否能够通信，并说明原因。
	192.168.2.0/24的表项的协议是RIP，Pre和Cost的值也发生了变化。
	能够ping通，直接原因是R1的路由表中有了该网络的相关表项，而该表项来自于RIP协议。
5. 写出实验中在路由器R1上配置静态路由、缺省路由和RIP协议所用的基本命令。

|       |                                             |
| ----- | ------------------------------------------- |
| 静态路由  | Ip route-static 192.168.2.0 24 192.168.1.1  |
| 缺省路由  | Ip route-static 0.0.0.0 0.0.0.0 192.168.1.1 |
| RIP协议 | Network 192.168.1.0                         |

6. 在路由器上，缺省路由也是一种静态路由，请说明为什么IP route-static 0.0.0.0  0.0.0.0 192.168.1.1表示缺省路由？
	因为该网络前缀长度为0，也就是说任何的目的IP都能够与之匹配上。当一个目标网络没有找到匹配的表项后，它一定能够匹配上该条路由，如此它便能够表明缺省路由。
7. 实验中，路由器在启动了RIP以后，下面命令是什么含义。`[R1-rip]network 192.168.1.0`？
该命令的含义是在路由器1的网段192.168.1.0上启动RIP。
8. 根据所截获的RIP响应报文，填写下表：观察所截取到的响应报文，填写下表：
	![07](https://hackmd.io/_uploads/HkL7LkOG-e.png)

9. 观察截取的RIP协议报文，请说明RIP协议是否只能用于TCP/IP网络，为什么？
RIP 协议**不局限于** **TCP/IP** **网络**，核心依据是其报文结构中 “网络协议簇字段” 的灵活设计（支持多协议类型标识），且协议核心的距离矢量算法不依赖 TCP/IP 的特定机制。TCP/IP 网络是 RIP 的主流应用场景，但并非唯一场景，它可通过适配不同协议簇的传输层机制，为其他网络协议（如 IPX、AppleTalk）提供路由信息交换服务。

## 5 距离矢量算法实验

实验内容：在计算机上用Wireshark截取RIP报文，分析距离矢量算法的计算过程。

### 实验步骤

步骤1 按照课本图6-8所示的组网图连接好设备，配置各设备的IP地址。注意，在路由器和三层交换机上都可以配置Loopback接口，Loopback是一种纯软件性质的虚拟接口，Loopback接口一旦被创建，将一直保持Up状态，直到被删除。

配置交换机S1

```
<H3C>system-view
[H3C]sysname S1
[S1]vlan 1
[S1-vlan1]port ge1/0/1 ge1/0/2 ge1/0/13
[S1-vlan1]quit
[S1]interface vlan 1
[S1-Vlan-interface1]ip addr 192.168.2.1 255.255.255.0
```

在S1上配置Loopback接口（平台上某些设备的接口配置回环地址时子网掩码都要求是32位，即255.255.255.255）

```
[S1-Vlan-interface1]quit
[S1]interface loopback 1
[S1-LoopBack1]ip address 192.168.1.1 255.255.255.255
```

配置交换机S2

```
<H3C>system-view
[H3C]sysname S2
[S2]vlan 1
[S2-vlan1]port ge1/0/1 ge1/0/2 ge1/0/13
[S2-vlan1]quit
[S2]interface vlan 1
[S2-Vlan-interface1]ip addr 192.168.3.2 255.255.255.0
```

配置路由器R1

```
<Router>system
[Router]sysname R1
[R1]interface ge 0/0
[R1-GigabitEthernet0/0]ip addr 192.168.2.2 255.255.255.0
[R1-GigabitEthernet0/0]quit
[R1]interface ge 0/1
[R1-GigabitEthernet0/1]ip addr 192.168.3.1 255.255.255.0
```

配置其余四台电脑IP地址

PCA

192.168.2.10

255.255.255.0

192.168.2.1

PCB

192.168.2.11

255.255.255.0

192.168.2.1

PCC

192.168.3.10

255.255.255.0

192.168.3.2

PCD

192.168.3.11

255.255.255.0

192.168.3.2

步骤2 在PCA或PCB上运行Wireshark，然后在各三层交换机和路由器上配置RIP。

配置路由器R1

```
[R1-GigabitEthernet0/1]quit
[R1]rip
[R1-rip-1]network 192.168.2.0
[R1-rip-1]network 192.168.3.0
```

配置交换机S1

```
[S1-LoopBack1]quit
[S1]rip
[S1-rip-1]network 192.168.2.0
[S1-rip-1]network 192.168.1.0 // 存疑 可能是192.168.1.1
```

配置交换机S2

```
[S2-Vlan-interface1]quit
[S2]rip
[S2-rip-1]network 192.168.3.0
```

查看S2的路由表

```
[S2]display ip routing-table
```

（截图）

步骤3 分析PCA上Wireshark截取的报文，选择一条Response RIPv1(source为192.168.2.1)报文，展开其中的`Routing Information Protocol`项和其中的`IP Address`项（并截图）

查看R1的路由表

```
[R1]display ip routing-table
```

（截图）

步骤4 在PCC或PCD上运行Wireshark抓取报文，观察路由器R1广播的报文(source为192.168.3.1)。

展开报文中的Routing Information信息和其中的IP Address信息

（截图）

查看S2的路由表

```
[S2]display ip routing-table
```

（截图）

### 实验报告增量

在S2上也配置一下Loopback地址，IP地址为192.168.4.1/24，通过RIP协议进行广播

```
[S2]interface loopback 1
[S2-Loopback1]ip add 192.168.4.1 255.255.255.255
[S2-Loopback1]quit
[S2]rip // 存疑
[S2-rip-1]network 192.168.4.0 // 存疑
```

观察并记下在R1和S1的路由表中关于该网段的路由条目

```
[R1]display ip routing-table
```

（截图）

```
[S1]display ip routing-table
```

（截图）

实验结束可以考虑帮队友取消一下S2的环回地址

```
[S2-rip-1]quit
[S2]undo interface loopback 1
```

## 6 触发更新和水平分割实验

### 6.1 实验步骤

##### 1
1. 继续上一小节的实验步骤，在 PCA 或 PCB 以及 PCC 或 PCD 上打开 wireshark，截取报文。
2. 取消交换机 S1 的回环地址 192.168.1.1,相当于 S1 到 192.168.1.1 网段的连接中断,观察 PCA 上截取到的报文。
	取消回环地址的命令:
	`[S1]undo interface loopback 1`
3. PCA 上观察到当取消S1的 Loopback 接口时,S1 立即产生一个 RIP 广播报文,目的地址为 192.168.1.0,跳数为 16,表示到网络 192.168.1.0 不可达。RIP 报文只包含改变了的路由信息,所以只有一条路由信息。
	![08(1)](https://hackmd.io/_uploads/rJAjIyOzWx.png)


##### 2 观察在 PCC 或 PCD 上截取的报文

由下图可见,紧接着 R1 收到该消息后也广播该信息,同样只包含一条改变了的路由信息,目的地址是 192.168.1.0,跳数为 16。这样,192.168.1.0 网段不可达的信息很快通知到自治系统内的所有路由器。
![09](https://hackmd.io/_uploads/BJvpUJdfbl.png)


##### 3
1. RIP 配置后默认启动水平分割。
2. 重新配置好 S1 的 Loopback 地址,使各路由器运行 RIP,正常工作。
	1. 创建 Loopback 接口并配置 IP：
	    `[s1]inter loopback 1
	    `[s1-LoopBack1]ip add 192.168.1.1 255.255.255.255`
	2. 确认 / 配置 S1 的 RIP 进程，包含 Loopback 所在网段（应该不需要）
	    `[s1]rip`
	    `[s1-rip-1]network 192.168.1.0  # 关键：让S1通过RIP广播192.168.1.0网段路由`
	    `[s1-rip-1]network 192.168.2.0  # 补充：S1与R1连接的网段`
	3. 验证 RIP 正常（可选）：
	    在 R1 上查看路由表，确认 `192.168.1.0/24` 网段的 RIP 路由已存在：
	    `[R1]display ip routing-table  # 应显示
	    `# 192.168.1.0/24  RIP  100  1  192.168.2.1  Ethernet0/0  # Loopback网段的RIP路由（来自S1）`
	    `# 192.168.2.0/24  Direct  0  0  192.168.2.2  Ethernet0/0  # S1与R1直连网段（优先级最高）`
3. 取消路由器各接口的水平分割功能,R1 的参考命令如下;其他的设备类似。
	`[R1]inter ge 0/0`
	`[R1-GigabitEthernet0/0]undo rip split-horizon`
	`[R1]inter ge 0/1`
	`[R1-GigabitEthernet0/1]undo rip split-horizon`
4. 在 PCA 或 PCB 上运行 Wireshark 截取报文。

##### 4
1. 观察路由器 R1 从 ge0/0 接口发出的RIP报文。
2. 如下图所示,和没有取消水平分割时的报文相比,多了一条到192.168.1.0的选路信息,这是为什么呢?因为 R1 到 192.168.1.0 网段选路信息是从 ge0/0 接口得到的,如果启动了水平分割,则这条选路信息不会再从该接口发送出去。现在,取消了水平分割功能,所以,R1 会在各端口发送所有已知的选路信息。查看其他报文,可以发现其他路由器发送的报文也有类似现象。
	![10](https://hackmd.io/_uploads/rJ-CIkdz-e.png)


### 6.2 实验报告

12. 比较水平分割前后RIP报文的选路信息的不同，把你截取的一条报文写在下表中？
	（存疑）
	![11](https://hackmd.io/_uploads/Skc0LJ_fWe.png)


## 7 RIPv2 报文结构分析实验

实验内容：在路由器/三层交换机上配置RIPv2协议，在计算机上用Wireshark截取报文，分析RIPv2协议各字段的含义。

### 实验步骤

步骤1 保留之前的组网和配置。

在各设备上启动rip命令

```
[S1]rip
```

```
[R1]rip
```

```
[S2]rip
```

在各接口配置RIPv2，并配置报文认证模式是MD5认证。

配置R1

```
[R1]interface ge0/0
[R1-GigabitEthernet0/0]rip version 2
[R1-GigabitEthernet0/0]rip authentication-mode md5 rfc2082 plain buaa 1
[R1-GigabitEthernet0/0]quit
[R1]interface ge0/1
[R1-GigabitEthernet0/1]rip version 2
[R1-GigabitEthernet0/1]rip authentication-mode md5 rfc2082 plain buaa 1
```

（这里新设备部分命令可能进行了微调，比如可能这里不应该加plain，可以使用?指令逐词测试）

测试样例

```
[R1-GigabitEthernet0/0]rip authentication-mode md5 ?
rfc2082 RFC 2082 MD5 authentication packet format type
rfc2453 RFC 2453 MD5 authentication packet format type
[R1-GigabitEthernet0/0]rip authentication-mode md5 rfc2082 ?
STRING<1-53> xxxx c
xxxxxxx
[R1-GigabitEthernet0/0]rip authentication-mode md5 rfc2082 buaa ？
INTEGER<1-255> xxxxxx
[R1-GigabitEthernet0/0]rip authentication-mode md5 rfc2082 buaa 1 // 成功
```

配置S1

```
[S1]inter vlan 1
[S1-Vlan-interface1]rip version 2
[S1-Vlan-interface1]rip authentication-mode md5 rfc2082 buaa 1
[S1-Vlan-interface1]quit
[S1]interface loopback 1
[S1-Loopback1]rip version 2
[S1-Loopback1]rip authentication-mode md5 rfc2082 buaa 1
```

配置S2

```
[S2]inter vlan 1
[S2-Vlan-interface1]rip version 2
[S2-Vlan-interface1]rip authentication-mode md5 rfc2082 buaa 1
```

步骤2 观察截取的报文

（截图，找RIPv2的报文，展开Routing Information Protocol详细信息）

## 8 设计实验 2

### 8.1 实验步骤

##### 1 组网连接

![12](https://hackmd.io/_uploads/r1T1v1dMbx.png)


| 设备 1 | 设备 1 端口 | 设备 2 | 设备 2 端口 |
| ---- | ------- | ---- | ------- |
| S1   | E0/1    | R1   | E0/0    |
| S2   | E0/1    | R2   | E0/1    |
| S1   | E0/24   | PCB  | Eth0    |
| S2   | E0/24   | PCC  | Eth0    |
| R1   | S0/0    | R2   | S0/0    |

##### 2 VLAN 划分配置（S1 与 S2）

###### 1. 交换机 S1 配置

```bash
<H3C>sys  # 进入系统视图
[S1]vlan 2  # 创建VLAN2
[S1-vlan2]port E0/20 to E0/24  # 将E0/20-E0/24划入VLAN2（其余端口默认VLAN1）
[S1-vlan2]quit
```

###### 2. 交换机 S2 配置

```bash
<H3C>sys
[S2]vlan 2
[S2-vlan2]port E0/20 to E0/24  # E0/20-E0/24划入VLAN2（其余端口默认VLAN1）
[S2-vlan2]quit
```

##### 3 IP 地址配置

###### 1. 交换机 S1 接口 IP

```bash
[S1]interface Vlan-interface1  # 进入VLAN1接口
[S1-Vlan-interface1]ip address 192.168.1.2 255.255.255.0
[S1-Vlan-interface1]quit

[S1]interface Vlan-interface2  # 进入VLAN2接口
[S1-Vlan-interface2]ip address 192.168.3.1 255.255.255.0
[S1-Vlan-interface2]quit
```

###### 2. 交换机 S2 接口 IP

```bash
[S2]interface Vlan-interface1
[S2-Vlan-interface1]ip address 192.168.2.2 255.255.255.0
[S2-Vlan-interface1]quit

[S2]interface Vlan-interface2
[S2-Vlan-interface2]ip address 192.168.4.1 255.255.255.0
[S2-Vlan-interface2]quit
```

###### 3. 路由器 R1 接口 IP

```bash
<H3C>sys
[R1]interface Ethernet0  # 连接S1的E0接口
[R1-Ethernet0]ip address 192.168.1.1 255.255.255.0
[R1-Ethernet0]quit

[R1]interface Serial 0/0  # 连接R2的S0/0串口
[R1-Serial0/0]link-protocol ppp  # 串口默认封装PPP（必配）
[R1-Serial0/0]ip address 192.168.5.1 255.255.255.0
[R1-Serial0/0]quit
```

###### 4. 路由器 R2 接口 IP

```bash
<H3C>sys
[R2]interface Serial 0/0  # 连接R1的S0/0串口
[R2-Serial0/0]link-protocol ppp  # 串口默认封装PPP（必配）
[R2-Serial0/0]ip address 192.168.5.2 255.255.255.0
[R2-Serial0/0]quit

[R2]interface Ethernet1  # 连接S2的E0/1接口
[R2-Ethernet1]ip address 192.168.2.1 255.255.255.0
[R2-Ethernet1]quit
```

###### 5. 计算机 IP 配置

- **PCB**：IP=192.168.3.2/24，网关 = 192.168.3.1
- **PCC**：IP=192.168.4.2/24，网关 = 192.168.4.1

##### 4 RIP 协议配置（开启并宣告所有相关网段）

###### 1. 交换机 S1（需支持三层功能）

```bash
[S1]rip  # 启动RIP
[S1-rip-1]network 192.168.1.0  # 宣告VLAN1网段
[S1-rip-1]network 192.168.3.0  # 宣告VLAN2网段
[S1-rip-1]quit
```

###### 2. 交换机 S2（需支持三层功能）

```bash
[S2]rip
[S2-rip-1]network 192.168.2.0  # 宣告VLAN1网段
[S2-rip-1]network 192.168.4.0  # 宣告VLAN2网段
[S2-rip-1]quit
```

###### 3. 路由器 R1

```bash
[R1]rip
[R1-rip-1]network 192.168.1.0  # 宣告连接S1的网段
[R1-rip-1]network 192.168.5.0  # 宣告连接R2的直连网段
[R1-rip-1]quit
```

###### 4. 路由器 R2

```bash
[R2]rip
[R2-rip-1]network 192.168.2.0  # 宣告连接S2的网段
[R2-rip-1]network 192.168.5.0  # 宣告连接R1的直连网段
[R2-rip-1]quit
```

#### 5 静态路由配置

```bash
# 路由器 R1（访问R2侧网段，可选）
[R1]ip route-static 192.168.2.0 255.255.255.0 192.168.5.2
[R1]ip route-static 192.168.4.0 255.255.255.0 192.168.5.2
[R1]ip route-static 192.168.3.0 255.255.255.0 192.168.1.2

# 路由器 R2（访问R1侧网段，可选）
[R2]ip route-static 192.168.1.0 255.255.255.0 192.168.5.1
[R2]ip route-static 192.168.3.0 255.255.255.0 192.168.5.1
[R2]ip route-static 192.168.4.0 255.255.255.0 192.168.2.2

[S1]ip route-static 192.168.4.0 255.255.255.0 192.168.1.1

[S2]ip route-static 192.168.3.0 255.255.255.0 192.168.2.1
```

##### 6 全网互通验证

1. PCB 执行：`ping 192.168.4.2`（PCC），应回显成功；
2. PCC 执行：`ping 192.168.3.2`（PCB），应回显成功；
3. R1 执行：`ping 192.168.4.2`（PCC）、`ping 192.168.2.1`（R2），应成功；
4. R2 执行：`ping 192.168.3.2`（PCB）、`ping 192.168.1.1`（R1），应成功。

### 8.2 实验报告

#### 1. 交换机 S1（三层交换机）

- **VLAN 与接口 IP 配置**
    - 创建 VLAN 2，将 E0/20-E0/24 端口划入 VLAN 2（其余端口默认 VLAN 1）；
    - 配置 VLAN 1 接口 IP：192.168.1.2/24，VLAN 2 接口 IP：192.168.3.1/24。
- **RIP 配置**
    启动 RIP 进程 1，宣告自身直连网段（实现路由信息传递）：
    - 启用 RIP：`rip`
    - 宣告 192.168.1.0/24（VLAN 1，连接 R1）
    - 宣告 192.168.3.0/24（VLAN 2，连接 PCB）
- **静态路由**
    `ip route-static 192.168.4.0 255.255.255.0 192.168.1.1`（指向 R1，访问 PCC 网段）。

#### 2. 交换机 S2（三层交换机）

- **VLAN 与接口 IP 配置**
    - 创建 VLAN 2，将 E0/20-E0/24 端口划入 VLAN 2；
    - 配置 VLAN 1 接口 IP：192.168.2.2/24，VLAN 2 接口 IP：192.168.4.1/24。
- **RIP 配置**
    启动 RIP 进程 1，宣告自身直连网段：
    - 启用 RIP：`rip`
    - 宣告 192.168.2.0/24（VLAN 1，连接 R2）
    - 宣告 192.168.4.0/24（VLAN 2，连接 PCC）
- **静态路由**
    `ip route-static 192.168.3.0 255.255.255.0 192.168.2.1`（指向 R2，访问 PCB 网段）。

#### 3. 路由器 R1

- **接口 IP 与封装配置**
    - E0/0 接口（连接 S1 E0/1）：IP 192.168.1.1/24；
    - Serial 0/0 接口（连接 R2 S0/0）：封装 PPP 协议，IP 192.168.5.1/24。
- **RIP 配置**
    启动 RIP 进程 1，宣告直连网段：
    - 启用 RIP：`rip`
    - 宣告 192.168.1.0/24（E0/0 网段）
    - 宣告 192.168.5.0/24（Serial 0/0 网段）
- **静态路由**
    - `ip route-static 192.168.2.0 255.255.255.0 192.168.5.2`
    - `ip route-static 192.168.4.0 255.255.255.0 192.168.5.2`

#### 4. 路由器 R2

- **接口 IP 与封装配置**
    - Serial 0/0 接口（连接 R1 S0/0）：封装 PPP 协议，IP 192.168.5.2/24；
    - E0/1 接口（连接 S2 E0/1）：IP 192.168.2.1/24。
- **RIP 配置**
    启动 RIP 进程 1，宣告直连网段：
    - 启用 RIP：`rip`
    - 宣告 192.168.2.0/24（E0/1 网段）
    - 宣告 192.168.5.0/24（Serial 0/0 网段）
- **静态路由**
    - `ip route-static 192.168.1.0 255.255.255.0 192.168.5.1`
    - `ip route-static 192.168.3.0 255.255.255.0 192.168.5.1`

#### 5. 计算机 PCB/PCC

- **PCB**：IP 192.168.3.2/24，网关 192.168.3.1（S1 VLAN 2 接口）；
- **PCC**：IP 192.168.4.2/24，网关 192.168.4.1（S2 VLAN 2 接口）；
- 说明：无路由协议 / 静态路由配置，依赖网关设备的 RIP 路由实现跨网段通信。

#### 6. 核心说明

1. **互通原理**：全网以 RIP 协议为核心实现路由信息自动传递，三层交换机通过 VLAN 接口实现本地三层转发，路由器通过 PPP 封装实现串口链路连通，最终保障 PCB 与 PCC、各网络设备间的跨网段互通；
2. **路由优先级**：静态路由（优先级 60）优先级高于 RIP（优先级 100），若同时配置，设备优先使用静态路由；仅依赖 RIP 时，可省略所有静态路由配置；
3. **验证要点**：除 ping 测试外，可通过 `display ip routing-table` 查看各设备路由表，确认非直连网段路由条目为 RIP 学习（标记为 “R”），验证 RIP 协议生效。
