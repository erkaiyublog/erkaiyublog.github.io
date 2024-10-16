---
published: false
title: 计网知识点简记
tags: network CN
---
最近在图书馆看到一本《VPNS A BEGINNER'S GUIDE》，比较有趣。该书的前半部分用于简单罗列一些计算机网络相关的知识点，后半部分讲VPN相关知识。由于鄙人自知本科计网课程学得极度不扎实，于是打算借此机会回顾一遍相关知识点并用这篇博客记录。由于计算机网络相关知识点太多（各种协议），本篇不会尝试记录所有细节（我脑子也记不住），只是将对应的知识点写在对应的栏目下方便日后查阅，并简单概括一下自己的理解。日后应该会不断将本书外学到的相关知识也在这篇博客里补充，但愿它可以变得臃肿而有意义吧。

# 目录
* TOC
{:toc}

# 网络模型概览
## OSI
OSI指Open System Interconnection，即常说的7层网络模型。

![OSI模型](../images/posts/network/osi.png)

OSI模型的普及是由美国政府在上世纪90年代促成的。下面自底向上罗列一下每一层的作用概括以及相关协议名称。
1. **Physical Layer**: 传输介质所在，数据以bits形式流通，RJ-45规则声明了cable中使用的链接器以及引脚。Ethernet和IEEE 802.3在这一层定义了网线的使用规则（数据如何通过网线传输）。
2. **Data Link Layer**: 处理physical addressing，负责某段网线上传输的packets（例如一个packet需要经过多条网线到达目的地，而这一层只关心其中的每一段网线而非全程）。当然本层会涉及网络的拓扑结构，error control，frame sequencing和flow control。相关协议有Frame Relay, HDLC, PPP (Point to Point Protocol), FDDI, ATM (Asynchronous Transfer Mode), IEEE 802.3 (Ethernet使用), IEEE 802.5。
3. **Network Layer**: 处理logical addressing。涉及协议有IP, IPX (Internetwork Packet Exchange), BGP, OSPF, RIP (Routing Information Protocol)。
4. **Transport Layer**: 涉及connection oriented或connectionless的端到端传输，这层一般用segment称呼数据。由于是负责数据传输，会涉及flow control, multiplexing, error checking和recovery。相关协议有TCP, UDP, SPX (Sequence Packet Exchange)。
5. **Session Layer**: 也叫port layer。负责两台机器间通讯的建立和管理。对于上层的layer而言，本层不仅提供了数据传输，且提供了一些基于数据传输的服务，例如远程登录、远程文件传输。本层也提供synchronization，例如一个长达两小时的文件传输在一小时后因网络原因中断，synchronization会保证在下次重新传输时仅传输剩余部分数据。相关协议包括RPC, SQL, NetBIOS, ASP (AppleTalk Session Protocol)。
6. **Presentation Layer**: 确保了向上层提供的数据是上层所在的应用可读的（翻译官）。同时负责数据加密、压缩和不同格式字符的转换。相关的协议有GIF, JPEG, TIFF, ASCII, MPEG, HTML。
7. **Application Layer**: 负责应用软件与网络交互。相关协议有HTTP, FTP, NFS, SMTP, SNMP (Simple Network Management Protocol, DNS。

OSI的不同层间交互是通过SAP (Service Access Point)，上下层间一般以Layer N+1, Layer N代称，且抽象了一些术语用于描述上下层通信的数据或信息，涉及的概念一般与Control Information和Data Unit相关。主旨是下层为上层提供服务，传输的数据是由上至下逐层包裹。

# LAN与WAN
## LAN
Local Area Network。一般在一个LAN上的所有设备会由同一种cable连接。LAN相关的协议涉及OSI的最底两层，即Data Link和Physical。首先需要知道，IEEE将Data Link分为了上下两个sublayers，上面的是LLC (Logical Link Control)，下面的是MAC (Media Access Control)。由IEEE 802.2协议定义LLC的行为，LLC负责了network layer与下层的网卡间的交互。

Ethernet是LAN中绕不开的概念。Ethernet最早是作为broadcast的网络，确保成员可以随时将信息发送到任意地点。Ethernet的信息在媒介中传输时一般是使用CSMA/CD (Carrier Sense Multiple Access with Collision Detection)，即所有终端共享一个媒介，并通过Carrier Sense的操作在发送数据前监听当前媒介是否在被使用，并由Collision Detection来检验自己发出的数据是否与别人的相撞；另一种媒介传输的实现方式是IEEE 802.5，它采用了token ring的形式，持有token的终端可以使用媒介发送数据。在Ethernet的实现中，传输数据的frame格式有两种基本的标准，一种就叫Ethernet，另一种叫IEEE 802.3，后者现在用的更普遍。

事实上，CSMA/CD已经是较为古早的Ethernet技术了，它被应用于half-duplex Ethernet，当时的Ethernet是通过hub去broadcast，所以会有很多碰撞，现在的Ethernet都是full-duplex Ethernet，使用的是switch而非hub，不会有碰撞。

## WAN
Wide Area Network。由于LAN一般是私有的（例如学校，公司等），使不同LAN间相通信需要用到WAN。对于LAN而言，WAN就像一个云☁️，这个云提供了该LAN想通信的任意LAN。WAN的协议作用于OSI的最底三层，相关协议有X.25, Frame Relay, ATM (Asynchronous Transfer Mode)。

WAN内数据传输时需要通过switching来挑选较快的传输路径，此处的switching是指不同的设计思路，而非真实存在的定死的protocol。message switching是挑选整条message的路径，这种方法已经不再被使用，packet switching是将message拆分为packet逐个挑选路径，circuit switching是在sender和receiver间固定一条专门用于二者传输的circuit。

packet switching又分为datagram packet switching和virtual circuit packet switching。datagram packet switching是当前主流的实现方式，每个datagram的大小在几百到几千bytes之间，由于datagram packet switching无法保证所有的datagram都被送达，所以在收发端都需要一个PAD (Packet Assembler Disassember)。发送端的PAD会将message拆成datagrams并给每个datagram加上一个sequence number，而接收端的PAD可以根据这个sequence number来排列datagram。virtual circuit packet switching是指在发送message前，发送端和接收端先达成共识使用某条网络线路，然后将所有的packet通过这条线路传输，ATM (Asynchronous Transfer Mode)协议使用了这种传输设计。

