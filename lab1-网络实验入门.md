---
title: 【计网实验】lab1-网络实验入门
date: 2025-11-11 18:35:50
tags: 学习经验
categories:
- [学习经验, 计网实验]
---

### 0 说在前面

1. 我学艺不精，本次实验纯属临时抱佛脚，欢迎纠错
2. 每个实验开始时，记得先查看路由器和交换机的配置是否是初始清空状态（我到现在都不知道清空状态是什么样的，**建议不管三七二十一直接清空配置**），否则一定要先清空设备配置，以免以前实验的配置影响本次实验。下面是清空配置的代码
```
<XXXX>reset saved-configuration // 先删除已保存的配置
[Y/N] 选择y
<XXXX>reboot
[Y/N] 选择n
[Y/N] 选择y
```
3. 好用的快捷键 `Ctrl+G`: `display current-configuration`

### 1 可以参考的资料有什么？

1. [mooc 视频、预习题目](https://www.icourse163.org/learn/BUAA-1002744004)
	1. 其中“课件-北航远程在线计算机网络实验平台介绍和操作说明”有**实验平台的使用教程**和**第一次实验的部分教程**，建议提前观看
	2. **夯**，部分实验有实验操作录屏，可以直接照着做（但有的教程一点用都没有）
2. [常见问题共享文档](https://docs.qq.com/doc/DVXJTdkV1UFJFcWZz)
	1. 基本上用处不大，但需要的时候就特别需要（我在说啥）
3. 每次实验的 ppt、pdf、实验报告、**VMware**：ftp://10.111.1.29（以下简称 **ftp**）
	1. 注意：和其他网站在浏览器搜索不同，这个网址（？）需要在文件夹里的输入框搜索
	   ![01](https://hackmd.io/_uploads/Syvrt1_zZx.png)

	2. ppt、pdf 部分有用，实验报告非常有用

### 2 第一次上课前要做什么

1. [预约实验](https://network-lab.mooc.buaa.edu.cn/)
	1. 设备组 选什么应该不会影响，以防万一也可以参考[常见问题共享文档](https://docs.qq.com/doc/DVXJTdkV1UFJFcWZz)
	2. 操作系统 参考[常见问题共享文档](https://docs.qq.com/doc/DVXJTdkV1UFJFcWZz)
	3. 实验时间 建议直接约 4 h 拉满
	4. 助手人数 无所谓
2. 下载资料 ftp
	1. VMware（ftp 里面的安装包为 windows 特供，macOS 请另寻出路 [VMware Fusion](https://pan.baidu.com/s/1txYNV16Na08qEg6MQDBHPg?pwd=thuv)）
	2. ftp-download-本科生里的所有东西
3. 预习 [mooc 视频、预习题目](https://www.icourse163.org/learn/BUAA-1002744004)
	1. 课件-北航远程在线计算机网络实验平台介绍和操作说明：必看，有实验平台使用教程
	2. 课件-实验 1 网络实验入门：等做试验再看也来得及

### 3 实验怎么做

本次共 4 个实验
1. 路由器和交换机的基本配置
2. 报文分析软件使用
3. 简单组网实验（mooc 有视频，直接照着做）
4. 基于地址转换的组网实验(选做)（mooc 有视频，直接照着做，但需要修改R1的E0/1接口地址 和 地址池）
	接口地址和地址池在[常见问题共享文档](https://docs.qq.com/doc/DVXJTdkV1UFJFcWZz)的第 29 个问题
由于第 3、4 个有视频，此处仅对第 1、2 个实验进行说明（步骤全部参考自 mooc 视频/狼书/ppt）

#### 3.1 路由器和交换机的基本配置
1. 打开 pcc 的 r1.ht 和 pcd 的 r2.ht ![02](https://hackmd.io/_uploads/H1oLKydGZl.png)

2. 回车，出现 `<...>` 样子的头，说明可以进行输入
3. 按下 `Ctrl+G` 或输入 `display current-configuration`，看到下面内容就是我们需要的输出
	![03](https://hackmd.io/_uploads/r1HIY1_GWl.png)

4. 实验完成(\*´ω \`)人(´ω \` \*)

### 3.2 报文分析软件使用

1. 打开软件 连线组网，点击“主机联网”并确认
	步骤 1 和步骤 2 可以参考 [mooc 视频](https://www.icourse163.org/learn/BUAA-1002744004)的课件-北航远程在线计算机网络实验平台介绍和操作说明-连线组网的 3:20-4:26
	![04](https://hackmd.io/_uploads/H1VvKJdGbg.png)

2. 将 pca 网络连接的IP地址设置为自动获取
3. 打开软件 Wireshark
	1. 先选中“本地连接”，然后再点击 start。
		![05](https://hackmd.io/_uploads/BkhDtJ_fWg.png)

	2. 在 filter 栏输入 `ftp`
		![06](https://hackmd.io/_uploads/SyXutkdGWx.png)

4. 使用资源管理器访问FTP服务器（ftp://10.111.1.29）
5. 从Wireshark截获的报文中任意选一个ftp报文，并进行分析，填写表格
