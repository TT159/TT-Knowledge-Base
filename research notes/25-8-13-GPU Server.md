---
date: 2025-08-13
---
NLP GPU SERVER

找Ed要了GPU资源，大概如下。有两种途径，一个是CS NLP实验室里的一个GPU SERVER - ROSE。 可能很多人用。另一个是Ed自己在一个美国的科研机构上的资源，Ed作为PI申请到的，叫Chameleon

1. NLP Lab Server - Rose

Step 1: 登录 Lectura，学校的远程host server

CS Lab Lectura虚拟server系统:

CS username is: tiantan

In the run dialog enter 'cmd'.

Once open simply type: 'ssh tiantan@lectura.cs.arizona.edu' to begin an

SSH connection to Lectura

password: 1636TTwoxiangni!

Step 2: 从 Lectura 登录 Rose（GPU 服务器）

命令行输入：

ssh rose.cs.arizona.edu

若是第一次登录rose，会出现如下图提示：

![](file:///C:\Users\hp\AppData\Local\Temp\ksohtml22700\wps1.jpg) 

这是 第一次连接 Rose 服务器 时的正常现象。  
SSH 会提醒你这是一个新主机，它的身份（公钥指纹）还没存到你本地的 known_hosts 文件里。

直接输入： yes 然后回车

它会把这个服务器的指纹保存到你在 Lectura 的 ~/.ssh/known_hosts 里，这样下次登录 Rose 就不会再提示。

接下来它会要求你输入 Lectura 账号的密码（因为 Rose 共享同一个账号体系）。

输入时不会显示字符，这是正常的。

成功后你就进入 Rose 的 shell

若不成功，出现下图，不要慌，只是连接断开了，当然也可能是因为权限的问题。

![](file:///C:\Users\hp\AppData\Local\Temp\ksohtml22700\wps2.jpg) 

重新输入：

ssh rose.cs.arizona.edu

显示下图，则成功登录：

![](file:///C:\Users\hp\AppData\Local\Temp\ksohtml22700\wps3.jpg) 

图里的信息是登录后系统自动显示的状态摘要，主要内容解释如下：

1. 系统版本信息

Ubuntu 24.04.1 LTS：服务器运行的操作系统版本（长期支持版）。

Linux 6.8.0-1009-gcp x86_64：内核版本。

2. 系统资源状态

System load: 4.12  
当前系统负载（越高表示越忙，>服务器 CPU 核数 时表示比较繁忙）。

Usage of /home: 92.0% of 5.41TB  
/home 分区已经使用了 92% 的空间，快满了。你要注意别存太多大文件。

Memory usage: 29%  
内存使用率 29%，还比较空闲。

Swap usage: 99%  
交换分区（虚拟内存）几乎满了，这可能是因为有人在运行占用大量内存的任务。

Processes: 1330  
当前运行的进程数量。

Users logged in: 5  
当前有 5 个用户同时登录。

3. 其他信息

1 zombie process  
有一个僵尸进程（一般不影响使用）。

158 updates can be applied immediately  
系统有 158 个可用更新（你可能没权限更新）。

System restart required  
系统需要重启才能完成某些更新（管理员的事情）。

Step 3: 我们可以运行下面命令来看GPU配置

nvidia-smi

![](file:///C:\Users\hp\AppData\Local\Temp\ksohtml22700\wps4.jpg)


### GPU 信息

- **型号**：NVIDIA H100 PCIe（两块）
- **显存大小**：81,559 MiB（≈ 80 GB）
- **驱动版本**：550.163.01
- **CUDA 版本**：12.4

### 当前使用情况
1. **GPU 0**
    - 显存占用：896 MiB（其中 884 MiB 被一个 Python 进程使用）
    - GPU 利用率：16%（有人在用，但不高）

2. **GPU 1**
    - 显存占用：6 MiB（基本空闲）
    - GPU 利用率：0%（完全闲置）


### 建议

你完全可以直接用 **GPU 1** 来跑实验，因为：
- 它目前没人用（几乎空闲）。
- 显存和计算资源都充足。