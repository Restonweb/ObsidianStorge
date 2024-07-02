## Basic Information 基本信息

**SNMP - Simple Network Management Protocol** is a protocol used to monitor different devices in the network (like routers, switches, printers, IoTs...).  
SNMP - 简单网络管理协议是一种用于监控网络中不同设备（如路由器、交换机、打印机、物联网等）的协议。

Copy 复制

```
PORT    STATE SERVICE REASON                 VERSION
161/udp open  snmp    udp-response ttl 244   ciscoSystems SNMPv3 server (public)
```

SNMP also uses the port **162/UDP** for **traps**. These are data **packets sent from the SNMP server to the client without being explicitly requested**.  
SNMP 还使用端口 162/UDP 进行陷阱。这些是未经显式请求而从SNMP服务器发送到客户端的数据包。

### MIB

To ensure that SNMP access works across manufacturers and with different client-server combinations, the **Management Information Base (MIB)** was created. MIB is an **independent format for storing device information**. A MIB is a **text** file in which all queryable **SNMP objects** of a device are listed in a **standardized** tree hierarchy. It contains at **least one** `**Object Identifier**` **(**`**OID**`**)**, which, in addition to the necessary **unique address** and a **name**, also provides information about the type, access rights, and a description of the respective object MIB files are written in the `Abstract Syntax Notation One` (`ASN.1`) based ASCII text format. The **MIBs do not contain data**, but they explain **where to find which information** and what it looks like, which returns values for the specific OID, or which data type is used.  
为了确保SNMP访问在制造商之间以及不同的客户端-服务器组合中正常工作，创建了管理信息库（MIB）。MIB是一种独立的设备信息存储格式。MIB 是一个文本文件，其中设备的所有可查询 SNMP 对象都列在标准化的树层次结构中。它包含至少一个 `**Object Identifier**` （ `**OID**` ），除了必要的唯一地址和名称外，它还提供有关类型、访问权限和相应对象的描述的信息 MIB 文件以基于 `Abstract Syntax Notation One` （ `ASN.1` ） 的 ASCII 文本格式编写。MIB 不包含数据，但它们解释了在何处查找哪些信息及其外观、返回特定 OID 的值或使用哪种数据类型。

### OIDs

**Object Identifiers (OIDs)** play a crucial role. These unique identifiers are designed to manage objects within a **Management Information Base (MIB)**.  
对象标识符 （OID） 起着至关重要的作用。这些唯一标识符旨在管理管理信息库 （MIB） 中的对象。

The highest levels of MIB object IDs, or OIDs, are allocated to diverse standard-setting organizations. It is within these top levels that the framework for global management practices and standards is established.  
最高级别的 MIB 对象 ID （OID） 分配给不同的标准制定组织。正是在这些最高层中，建立了全球管理实践和标准的框架。

Furthermore, vendors are granted the liberty to establish private branches. Within these branches, they have the **autonomy to include managed objects pertinent to their own product lines**. This system ensures that there is a structured and organized method for identifying and managing a wide array of objects across different vendors and standards.  
此外，供应商可以自由地建立私人分支机构。在这些分支中，它们可以自主地包含与自己的产品线相关的托管对象。该系统确保有一种结构化和有组织的方法来识别和管理不同供应商和标准中的各种对象。

![](https://book.hacktricks.xyz/~gitbook/image?url=https:%2F%2F129538173-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-L_2uGJGU7AVNRcqRvEi%252Fuploads%252Fgit-blob-59fede678234060bdf9e0832b7ce4ed04bfd9bd6%252FSNMP_OID_MIB_Tree.png%3Falt=media&width=768&dpr=4&quality=100&sign=b851a4c4ba19b0aaa2e712c0a222ed71eeaf6fb9c14c5ab7624b02cc4bbe6e63)

You can **navigate** through an **OID tree** from the web here: [http://www.oid-info.com/cgi-bin/display?tree=#focus](http://www.oid-info.com/cgi-bin/display?tree=#focus) or **see what a OID means** (like `1.3.6.1.2.1.1`) accessing [http://oid-info.com/get/1.3.6.1.2.1.1](http://oid-info.com/get/1.3.6.1.2.1.1). There are some **well-known OIDs** like the ones inside [1.3.6.1.2.1](http://oid-info.com/get/1.3.6.1.2.1) that references MIB-2 defined Simple Network Management Protocol (SNMP) variables. And from the **OIDs pending from this one** you can obtain some interesting host data (system data, network data, processes data...)  
您可以在此处从 Web 浏览 OID 树： [http://www.oid-info.com/cgi-bin/display?tree=#focus](http://www.oid-info.com/cgi-bin/display?tree=#focus) 或查看 OID 的含义（如 `1.3.6.1.2.1.1` ）访问 [http://oid-info.com/get/1.3.6.1.2.1.1](http://oid-info.com/get/1.3.6.1.2.1.1) 。 有一些众所周知的 OID，例如 1.3.6.1.2.1 中的 OID，它引用了 MIB-2 定义的简单网络管理协议 （SNMP） 变量。从这个待处理的 OID 中，您可以获得一些有趣的主机数据（系统数据、网络数据、进程数据......

### **OID Example OID 示例**

[**Example from here**](https://www.netadmintools.com/snmp-mib-and-oids/): 这里的例子：

`**1 . 3 . 6 . 1 . 4 . 1 . 1452 . 1 . 2 . 5 . 1 . 3. 21 . 1 . 4 . 7**`

Here is a breakdown of this address.  
这是此地址的明细。

- 1 – this is called the ISO and it establishes that this is an OID. This is why all OIDs start with “1”  
    1 – 这称为 ISO，它确定这是一个 OID。这就是为什么所有 OID 都以“1”开头的原因
    
- 3 – this is called ORG and it is used to specify the organization that built the device.  
    3 – 这称为 ORG，用于指定构建设备的组织。
    
- 6 – this is the dod or the Department of Defense which is the organization that established the Internet first.  
    6 – 这是国防部或国防部，它是首先建立互联网的组织。
    
- 1 – this is the value of the internet to denote that all communications will happen through the Internet.  
    1 – 这是互联网的价值，表示所有通信都将通过互联网进行。
    
- 4 – this value determines that this device is made by a private organization and not a government one.  
    4 – 此值确定此设备是由私人组织而不是政府组织制造的。
    
- 1 – this value denotes that the device is made by an enterprise or a business entity.  
    1 – 此值表示设备由企业或商业实体制造。
    

These first six values tend to be the same for all devices and they give you the basic information about them. This sequence of numbers will be the same for all OIDs, except when the device is made by the government.  
对于所有设备，前六个值往往是相同的，它们为您提供了有关它们的基本信息。对于所有 OID，此数字序列都是相同的，除非设备由政府制造。

Moving on to the next set of numbers.  
继续下一组数字。

- 1452 – gives the name of the organization that manufactured this device.  
    1452 – 给出制造此设备的组织的名称。
    
- 1 – explains the type of device. In this case, it is an alarm clock.  
    1 – 解释设备类型。在这种情况下，它是一个闹钟。
    
- 2 – determines that this device is a remote terminal unit.  
    2 – 确定此设备是远程终端单元。
    

The rest of the values give specific information about the device.  
其余值提供有关设备的特定信息。

- 5 – denotes a discrete alarm point.  
    5 – 表示离散报警点。
    
- 1 – specific point in the device  
    1 – 设备中的特定点
    
- 3 – port 3 – 端口
    
- 21 – address of the port  
    21 – 端口地址
    
- 1 – display for the port  
    1 – 端口显示
    
- 4 – point number 4 – 点数
    
- 7 – state of the point  
    7 – 点的状态
    

### SNMP Versions SNMP 版本

There are 2 important versions of SNMP:  
SNMP 有 2 个重要版本：

- **SNMPv1**: Main one, it is still the most frequent, the **authentication is based on a string** (community string) that travels in **plain-text** (all the information travels in plain text). **Version 2 and 2c** send the **traffic in plain text** also and uses a **community string as authentication**.  
    SNMPv1：主要的，它仍然是最常见的，身份验证基于以纯文本形式传输的字符串（社区字符串）（所有信息都以纯文本形式传输）。 版本 2 和 2c 也以纯文本形式发送流量，并使用社区字符串作为身份验证。
    
- **SNMPv3**: Uses a better **authentication** form and the information travels **encrypted** using (**dictionary attack** could be performed but would be much harder to find the correct creds than in SNMPv1 and v2).  
    SNMPv3：使用更好的身份验证形式，并且使用加密信息传输（可以执行字典攻击，但比 SNMPv1 和 v2 更难找到正确的信任）。
    

### Community Strings 社区字符串

As mentioned before, **in order to access the information saved on the MIB you need to know the community string on versions 1 and 2/2c and the credentials on version 3.** The are **2 types of community strings**:  
如前所述，为了访问保存在MIB上的信息，您需要知道版本1和2/2c上的社区字符串以及版本3上的凭据。 社区字符串有 2 种类型：

- `**public**` mainly **read only** functions  
    `**public**` 主要为只读函数
    
- `**private**` **Read/Write** in general  
    `**private**` 一般读/写
    

Note that **the writability of an OID depends on the community string used**, so **even** if you find that "**public**" is being used, you could be able to **write some values.** Also, there **may** exist objects which are **always "Read Only".** If you try to **write** an object a `**noSuchName**` **or** `**readOnly**` **error** is received**.**  
请注意，OID 的可写性取决于所使用的社区字符串，因此即使您发现正在使用“public”，您也可以编写一些值。 此外，可能存在始终为“只读”的对象。 如果尝试写入对象，则会收到 `**noSuchName**` 或 `**readOnly**` 错误**。

In versions 1 and 2/2c if you to use a **bad** community string the server wont **respond**. So, if it responds, a **valid community strings was used**.  
在版本 1 和 2/2c 中，如果使用错误的社区字符串，服务器将不会响应。因此，如果它响应，则使用有效的社区字符串。

## Ports

[From Wikipedia](https://en.wikipedia.org/wiki/Simple_Network_Management_Protocol#:~:text=All%20SNMP%20messages%20are%20transported,port%20161%20in%20the%20agent.): 来自维基百科 ：

- The SNMP agent receives requests on UDP port **161**.  
    SNMP 代理在 UDP 端口 161 上接收请求。
    
- The manager receives notifications ([Traps](https://en.wikipedia.org/wiki/Simple_Network_Management_Protocol#Trap) and [InformRequests](https://en.wikipedia.org/wiki/Simple_Network_Management_Protocol#InformRequest)) on port **162**.  
    管理器在端口 162 上接收通知（Traps 和 InformRequests）。
    
- When used with [Transport Layer Security](https://en.wikipedia.org/wiki/Transport_Layer_Security) or [Datagram Transport Layer Security](https://en.wikipedia.org/wiki/Datagram_Transport_Layer_Security), requests are received on port **10161** and notifications are sent to port **10162**.  
    当与传输层安全性或数据报传输层安全性一起使用时，将在端口 10161 上接收请求，并将通知发送到端口 10162。
    

## Brute-Force Community String (v1 and v2c)  

To **guess the community string** you could perform a dictionary attack. Check [here different ways to perform a brute-force attack against SNMP](https://book.hacktricks.xyz/generic-methodologies-and-resources/brute-force#snmp). A frequently used community string is `public`.  
要猜测社区字符串，您可以执行字典攻击。在这里查看对SNMP执行暴力攻击的不同方法。常用的社区字符串是 `public` 。

## Enumerating SNMP 枚举 SNMP

It is recommanded to install the following to see whats does mean **each OID gathered** from the device:  
建议安装以下内容，以查看从设备收集的每个 OID 的含义：

Copy 复制

```
apt-get install snmp-mibs-downloader
download-mibs
# Finally comment the line saying "mibs :" in /etc/snmp/snmp.conf
sudo vi /etc/snmp/snmp.conf
```

If you know a valid community string, you can access the data using **SNMPWalk** or **SNMP-Check**:  
如果您知道有效的社区字符串，则可以使用 SNMPWalk 或 SNMP-Check 访问数据：

Copy 复制

```
snmpbulkwalk -c [COMM_STRING] -v [VERSION] [IP] . #Don't forget the final dot
snmpbulkwalk -c public -v2c 10.10.11.136 .

snmpwalk -v [VERSION_SNMP] -c [COMM_STRING] [DIR_IP]
snmpwalk -v [VERSION_SNMP] -c [COMM_STRING] [DIR_IP] 1.3.6.1.2.1.4.34.1.3 #Get IPv6, needed dec2hex
snmpwalk -v [VERSION_SNMP] -c [COMM_STRING] [DIR_IP] NET-SNMP-EXTEND-MIB::nsExtendObjects #get extended
snmpwalk -v [VERSION_SNMP] -c [COMM_STRING] [DIR_IP] .1 #Enum all

snmp-check [DIR_IP] -p [PORT] -c [COMM_STRING]

nmap --script "snmp* and not snmp-brute" <target>

braa <community string>@<IP>:.1.3.6.* #Bruteforce specific OID
```

Thanks to extended queries (download-mibs), it is possible to enumerate even more about the system with the following command :  
借助扩展查询 （download-mibs），可以使用以下命令枚举有关系统的更多信息：

Copy 复制

```
snmpwalk -v X -c public <IP> NET-SNMP-EXTEND-MIB::nsExtendOutputFull
```

**SNMP** has a lot of information about the host and things that you may find interesting are: **Network interfaces** (IPv4 and **IPv6** address), Usernames, Uptime, Server/OS version, and **processes**  
SNMP有很多关于主机的信息，您可能会发现有趣的东西是：网络接口（IPv4和IPv6地址），用户名，正常运行时间，服务器/操作系统版本和进程

**running** (may contain passwords)....  
正在运行（可能包含密码）...。

### **Dangerous Settings 危险设置**

In the realm of network management, certain configurations and parameters are key to ensuring comprehensive monitoring and control.  
在网络管理领域，某些配置和参数是确保全面监测和控制的关键。

### Access Settings 访问设置

Two main settings enable access to the **full OID tree**, which is a crucial component in network management:  
两个主要设置允许访问完整的 OID 树，这是网络管理中的关键组件：

1. `**rwuser noauth**` is set to permit full access to the OID tree without the need for authentication. This setting is straightforward and allows for unrestricted access.  
    `**rwuser noauth**` 设置为允许对 OID 树进行完全访问，而无需进行身份验证。此设置简单明了，允许不受限制的访问。
    
2. For more specific control, access can be granted using:  
    对于更具体的控制，可以使用以下方式授予访问权限：
    
    - `**rwcommunity**` for **IPv4** addresses, and  
        `**rwcommunity**` 表示 IPv4 地址，以及
        
    - `**rwcommunity6**` for **IPv6** addresses.  
        `**rwcommunity6**` 表示 IPv6 地址。
        
    

Both commands require a **community string** and the relevant IP address, offering full access irrespective of the request's origin.  
这两个命令都需要社区字符串和相关的 IP 地址，无论请求的来源如何，都提供完全访问权限。

### SNMP Parameters for Microsoft Windows  
Microsoft Windows 的 SNMP 参数

A series of **Management Information Base (MIB) values** are utilized to monitor various aspects of a Windows system through SNMP:  
一系列管理信息库 （MIB） 值用于通过 SNMP 监视 Windows 系统的各个方面：

- **System Processes**: Accessed via `1.3.6.1.2.1.25.1.6.0`, this parameter allows for the monitoring of active processes within the system.  
    系统进程：通过 `1.3.6.1.2.1.25.1.6.0` 访问，此参数允许监视系统内的活动进程。
    
- **Running Programs**: The `1.3.6.1.2.1.25.4.2.1.2` value is designated for tracking currently running programs.  
    正在运行的程序： `1.3.6.1.2.1.25.4.2.1.2` 值指定用于跟踪当前正在运行的程序。
    
- **Processes Path**: To determine where a process is running from, the `1.3.6.1.2.1.25.4.2.1.4` MIB value is used.  
    进程路径：要确定进程的运行位置，请使用 `1.3.6.1.2.1.25.4.2.1.4` MIB 值。
    
- **Storage Units**: The monitoring of storage units is facilitated by `1.3.6.1.2.1.25.2.3.1.4`.  
    存储单元：存储单元的监控由 `1.3.6.1.2.1.25.2.3.1.4` 提供。
    
- **Software Name**: To identify the software installed on a system, `1.3.6.1.2.1.25.6.3.1.2` is employed.  
    软件名称：为了标识系统上安装的软件，请使用 `1.3.6.1.2.1.25.6.3.1.2` 。
    
- **User Accounts**: The `1.3.6.1.4.1.77.1.2.25` value allows for the tracking of user accounts.  
    用户帐户： `1.3.6.1.4.1.77.1.2.25` 值允许跟踪用户帐户。
    
- **TCP Local Ports**: Finally, `1.3.6.1.2.1.6.13.1.3` is designated for monitoring TCP local ports, providing insight into active network connections.  
    TCP 本地端口：最后， `1.3.6.1.2.1.6.13.1.3` 被指定用于监视 TCP 本地端口，提供对活动网络连接的洞察。
    

### Cisco 思科

Take a look to this page if you are Cisco equipment:  
如果您是思科设备，请查看此页面：

[PAGE 页Cisco SNMP 思科SNMP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp/cisco-snmp)

## From SNMP to RCE 从SNMP到RCE

If you have the **string** that allows you to **write values** inside the SNMP service, you may be able to abuse it to **execute commands**:  
如果您有允许您在 SNMP 服务中写入值的字符串，则可以滥用它来执行命令：

[PAGE 页SNMP RCE SNMP RCE的](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp/snmp-rce)

## **Massive SNMP 海量SNMP**

[Braa](https://github.com/mteg/braa) is a mass SNMP scanner. The intended usage of such a tool is, of course, making SNMP queries – but unlike snmpwalk from net-snmp, it is able to query dozens or hundreds of hosts simultaneously, and in a single process. Thus, it consumes very few system resources and does the scanning VERY fast.  
Braa是一款大规模SNMP扫描仪。当然，这种工具的预期用途是进行SNMP查询 - 但与net-snmp的snmpwalk不同，它能够同时查询数十或数百个主机，并且在一个进程中。因此，它消耗很少的系统资源，并且扫描速度非常快。

Braa implements its OWN snmp stack, so it does NOT need any SNMP libraries like net-snmp.  
Braa 实现了自己的 snmp 堆栈，因此它不需要任何 SNMP 库，例如 net-snmp。

**Syntax:** braa [Community-string]@[IP of SNMP server]:[iso id]  
语法：braa [Community-string]@[SNMP服务器的IP]：[iso id]

Copy 复制

```
braa ignite123@192.168.1.125:.1.3.6.*
```

This can extract a lot MB of information that you cannot process manually.  
这可以提取大量无法手动处理的信息。

So, lets look for the most interesting information (from [https://blog.rapid7.com/2016/05/05/snmp-data-harvesting-during-penetration-testing/](https://blog.rapid7.com/2016/05/05/snmp-data-harvesting-during-penetration-testing/)):  
因此，让我们寻找最有趣的信息（来自 [https://blog.rapid7.com/2016/05/05/snmp-data-harvesting-during-penetration-testing/](https://blog.rapid7.com/2016/05/05/snmp-data-harvesting-during-penetration-testing/) ）：

### **Devices 设备**

The process begins with the extraction of **sysDesc MIB data** (1.3.6.1.2.1.1.1.0) from each file to identify the devices. This is accomplished through the use of a **grep command**:  
该过程从每个文件中提取 sysDesc MIB 数据 （1.3.6.1.2.1.1.1.1.0） 以识别设备。这是通过使用 grep 命令来实现的：

Copy 复制

```
grep ".1.3.6.1.2.1.1.1.0" *.snmp
```

### **Identify Private String 识别私有字符串**

A crucial step involves identifying the **private community string** used by organizations, particularly on Cisco IOS routers. This string enables the extraction of **running configurations** from routers. The identification often relies on analyzing SNMP Trap data for the word "trap" with a **grep command**:  
一个关键步骤涉及识别组织使用的专用社区字符串，特别是在Cisco IOS路由器上。此字符串允许从路由器提取正在运行的配置。识别通常依赖于使用 grep 命令分析单词“trap”的 SNMP 陷阱数据：

Copy 复制

```
grep -i "trap" *.snmp
```

### **Usernames/Passwords 用户名/密码**

Logs stored within MIB tables are examined for **failed logon attempts**, which might accidentally include passwords entered as usernames. Keywords such as _fail_, _failed_, or _login_ are searched to find valuable data:  
将检查存储在 MIB 表中的日志是否存在失败的登录尝试，其中可能会意外地包含作为用户名输入的密码。搜索 fail、failed 或 login 等关键字以查找有价值的数据：

Copy 复制

```
grep -i "login\|fail" *.snmp
```

### **Emails 电子邮件**

Finally, to extract **email addresses** from the data, a **grep command** with a regular expression is used, focusing on patterns that match email formats:  
最后，为了从数据中提取电子邮件地址，使用带有正则表达式的 grep 命令，重点关注与电子邮件格式匹配的模式：

Copy 复制

```
grep -E -o "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,6}\b" *.snmp
```

## Modifying SNMP values 修改SNMP值

You can use _**NetScanTools**_ to **modify values**. You will need to know the **private string** in order to do so.  
可以使用 NetScanTools 修改值。为此，您需要知道私有字符串。

## Spoofing 欺骗

If there is an ACL that only allows some IPs to query the SMNP service, you can spoof one of this addresses inside the UDP packet an sniff the traffic.  
如果存在仅允许某些 IP 查询 SMNP 服务的 ACL，则可以在 UDP 数据包中欺骗其中一个地址并嗅探流量。

## Examine SNMP Configuration files 检查 SNMP 配置文件

- snmp.conf
    
- snmpd.conf
    
- snmp-config.xml
    

![](https://book.hacktricks.xyz/~gitbook/image?url=https:%2F%2F129538173-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-L_2uGJGU7AVNRcqRvEi%252Fuploads%252FwdlXOpyZOVGNzyhOiiFK%252Fimage%2520%281%29.png%3Falt=media%26token=13f4d279-7d3f-47ce-a68e-35f9a906973f&width=768&dpr=4&quality=100&sign=4e496376a231b18ad4998087c113675290ac33234116f7f2d1cac1e6f43deb69)

If you are interested in **hacking career** and hack the unhackable - **we are hiring!** (_fluent polish written and spoken required_).  
如果您对黑客职业感兴趣并破解不可破解 - 我们正在招聘！ （需要流利的波兰语书面和口语）。

[![Logo](https://static.wixstatic.com/media/3143c3_922aaf9c0eca4b53aa244344583c598f%7Emv2.png/v1/fill/w_32%2Ch_32%2Clg_1%2Cusm_0.66_1.00_0.01/3143c3_922aaf9c0eca4b53aa244344583c598f%7Emv2.png)Careers | stmcyber.com | penetration testing  
招贤纳士 |stmcyber.com |渗透测试stmcyber.com](https://www.stmcyber.com/careers)

## HackTricks Automatic Commands HackTricks 自动命令

Copy 复制

```
Protocol_Name: SNMP    #Protocol Abbreviation if there is one.
Port_Number:  161     #Comma separated if there is more than one.
Protocol_Description: Simple Network Managment Protocol         #Protocol Abbreviation Spelled out

Entry_1:
  Name: Notes
  Description: Notes for SNMP
  Note: |
    SNMP - Simple Network Management Protocol is a protocol used to monitor different devices in the network (like routers, switches, printers, IoTs...).

    https://book.hacktricks.xyz/pentesting/pentesting-snmp

Entry_2:
  Name: SNMP Check
  Description: Enumerate SNMP
  Command: snmp-check {IP}

Entry_3:
  Name: OneSixtyOne
  Description: Crack SNMP passwords
  Command: onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings-onesixtyone.txt {IP} -w 100

Entry_4:
  Name: Nmap
  Description: Nmap snmp (no brute)
  Command: nmap --script "snmp* and not snmp-brute" {IP}

Entry_5:
  Name: Hydra Brute Force
  Description: Need Nothing
  Command: hydra -P {Big_Passwordlist} -v {IP} snmp
  
  
`