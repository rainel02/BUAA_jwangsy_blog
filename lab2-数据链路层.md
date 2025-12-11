---
title: 【计网实验】lab2-数据链路层
date: 2025-11-18 21:21:56
tags: 学习经验
categories:
- [学习经验, 计网实验]
---

本此实验实验内容
- 以太网链路层帧格式分析
- 交换机的 MAC 地址表和端口聚合
- VLAN 的配置与tag分析（hybrid选做）
- PPP协议分析及配置
- PPP认证（PAP必做，CHAP选做）

注：

1. 目前仅更新了前两个实验，剩下的什么时候更新就得看我的编译了……哈哈 O(∩_∩)O 
2. 不过本次实验除了hybrid外都有视频教程，而且hybrid的那个实验也不难（就是pdf和ppt都得参考一下），问题不大，各位实验加油ヾ(◍°∇°◍)ﾉﾞ
3. 下面用横线划掉的内容都是实验报告的相关内容，不是没用的东西
4. 刚刚登了一下小程序，发现下午改的密码又没用了。朋友！一周了！这bug都没搞定吗……而且今天下午上课的时候小程序也bug不断……建议和软工联动一下好吧，包给你交上各种各样好看实用的小程序好吧

## 1 以太网链路层帧格式分析

##### 实验目的

分析比较 Ethernet Ⅱ标准和 IEEE 802.3 标准的 MAC 帧格式

##### 实验步骤说明

1. 按照组网图组网并配置各设备；
	1. 组网
	   ![1](https://hackmd.io/_uploads/HyfqOJOf-e.png)

	2. 清空配置
	   `<XXXX>reset saved-configuration // 先删除已保存的配置`
	   `[Y/N] 选择 y`
	   `<XXXX>reboot`
	   `[Y/N] 选择 n`
	   `[Y/N] 选择 y`
2. Ethernet Ⅱ相关操作
	1. 配置 PCA 的 IP 地址：192.168.1.22，子网掩码 24 位
	2. 配置 PCB 的 IP 地址：192.168.1.21，子网掩码 24 位
	3. 在 PCA 和 PCB 上启动 Wireshark 软件；（我觉得 PCB 不需要启动）
	   本地连接-start
	4. 截获 Ethernet Ⅱ标准的 MAC 帧：
		1. 在 PCA  ping  PCB：cmd 上 `ping 192.168.1.21`
		2. 停止截获报文，保存并分析；
		   可以看到使用的是Ethernet V2标准，~~选中的报文缺少前导符、起始符、数据校验字段。原因如下：在抓包的时候，前导符和起始符已经被确认过，并且数据也被校检完毕。在这个时刻，前导符、起始符、数据校验字段没有必要保留，因此系统将它们丢弃，我们的报文中就缺少了那些数据字段。只留下数字链路层所需要的字段：目的MAC地址、源MAC地址、类型以及数据字段。~~
3. IEEE 802.3相关操作
	1. 配置 PCA
	   ![2](https://hackmd.io/_uploads/rkXsOkuMWl.png)

	2. 启动抓包软件
	   ![3](https://hackmd.io/_uploads/B1g3OkOM-e.png)

	   可以看到其使用的是IEEE 802.3标准的MAC帧，其类型字段已变成长度字段。

## 2 交换机的 MAC 地址表和端口聚合

##### 实验目的

- 理解交换机 MAC 地址的学习过程
- 理解端口聚合的原理（这个不做）

##### 实验步骤-交换机的 MAC 地址表

1. 继续上节实验，依旧如图组网，重新配置 PCA 的 IP 地址：192.168.1.22，在计算机 PCA 上执行 ping 192.168.1.21。
   ![1](https://hackmd.io/_uploads/rk26Okdf-e.png)

2. 查看交换机的 MAC 地址表
	`<Quidway>sys`
	`[Quidway]dis mac`
	- ~~结果为~~
	  ![4](https://hackmd.io/_uploads/HyB0uyOMbe.png)

	- 解释 MAC 地址表中各字段的含义：
	  ~~第一列是MAC地址，代表设备的MAC地址；
	  第四列是端口号，代表相应MAC地址是通过该端口学来的，即这个MAC地址的设备是连接在这个端口上的，但可能不是直接相连；
	  第二列是VLAN编号，代表这个端口所属的VLAN编号；
	  第三列是STATE，代表这条信息是怎么得来的，学来的还是配置得来的；
	  第五列是生命值，以秒为单位，对于学来的，需要定期遗忘，如果一直没得到刷新，当AGING归零后该信息将被丢弃。~~
	- 这个实验能够说明 MAC 地址表的学习是来源于源 MAC 地址而非目的 MAC 地址？试给出一个验证方法。
	  ~~来源于源 MAC 地址~~
		1. 超级终端里
		   `[Quidway]undo mac` `y`
		   `[Quidway]dis mac`
		   ![5](https://hackmd.io/_uploads/BkpRu1uGZl.png)

		   发现交换机又学来了 2 条信息，这两条信息究竟是从哪里学到的呢？我们需要看一下主机们的 mac 地址
		2. ~~查看主机们的 mac 地址
		   cmd 上输出 `ipconfig /all~~
		   ![6](https://hackmd.io/_uploads/rJzgYkdfbx.png)![7](https://hackmd.io/_uploads/H1_etJ_GWg.png)
		   发现刚刚交换机学的那两条信息不是主机 A 的也不是主机 B 的，而是交换机自身的
		3. ~~再次清除MAC地址表并ping
		   `[Quidway]undo mac` `y`
		   `[Quidway]dis mac` 发现已经地址表已经全空
		   断开 PCB 的连线，然后cmd 上 `ping 192.168.1.21`
		   `[Quidway]dis mac` 发现学到了主机 A 的 mac 地址但没发现主机 B 的 mac 地址~~
		   ![8](https://hackmd.io/_uploads/B1FkFkdfWg.png)

