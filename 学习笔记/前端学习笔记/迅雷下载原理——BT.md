# 迅雷下载原理——BT

[toc]

## 原理概述

[原理动画]: http://mg8.org/processing/bt.html

普通的HTTP／FTP下载使用TCP/IP协议，BitTorrent协议是架构于TCP/IP协议之上的一个P2P文件传输通信协议，处于TCP/IP结构的应用层。BitTorrent协议本身也包含了很多具体的内容协议和扩展协议，并在不断扩充中。根据BitTorrent协议，文件发布者会根据要发布的文件生成提供一个**.torrent文件**，即种子文件，也简称为“种子”。

种子文件本质上是文本文件，包含**Tracker信息**和**文件信息**两部分。Tracker信息主要是BT下载中需要用到的**Tracker服务器**的地址和针对Tracker服务器的设置，文件信息是根据对目标文件的计算生成的，计算结果根据BitTorrent协议内的Bencode规则进行编码。它的主要原理是需要把提供下载的文件虚拟分成大小相等的块，块大小必须为2k的整数次方（由于是虚拟分块，硬盘上并不产生各个块文件），并把每个块的索引信息和Hash验证码写入种子文件中；所以，种子文件就是被下载文件的“索引”。

下载者要下载文件内容，需要先得到相应的种子文件，然后使用BT客户端软件进行下载。下载时，BT客户端首先解析种子文件得到Tracker地址，然后连接Tracker服务器。Tracker服务器回应下载者的请求，提供下载者其他下载者（包括发布者）的IP。下载者再连接其他下载者，根据种子文件，两者分别告知对方自己已经有的块，然后交换对方所没有的数据。此时不需要其他服务器参与，分散了单个线路上的数据流量，因此减轻了服务器负担。

![img](https://upload-images.jianshu.io/upload_images/7734354-5ea12d7926db62c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

下载者每得到一个块，需要算出下载块的Hash验证码与种子文件中的对比，如果一样则说明块正确，不一样则需要重新下载这个块。这种规定是为了解决下载内容准确性的问题。

![img](https://upload-images.jianshu.io/upload_images/7734354-346a5f0d40231174.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/400/format/webp)

一般的HTTP/FTP下载，发布文件仅在某个或某几个服务器，下载的人太多，服务器的带宽很易不胜负荷，变得很慢。而BitTorrent协议下载的特点是，下载的人越多，提供的带宽也越多，下载速度就越快。同时，拥有完整文件的用户也会越来越多，使文件的“寿命”不断延长。