.. vim: syntax=rst

ETH—Lwip以太网通信
=====================

互联网技术对人类社会的影响不言而喻。当今大部分电子设备都能以不同的方式接入互联网(Internet)，在家庭中PC常见的互联网接入方式是使用路由器(Router)组建小型局域网(LAN)，
利用互联网专线或者调制调解器(modem)经过电话线网络，连接到互联网服务提供商(ISP)，由互联网服务提供商把用户的局域网接入互联网。
而企业或学校的局域网规模较大，常使用交换机组成局域网，经过路由以不同的方式接入到互联网中。

互联网模型
~~~~~~~~~~~~~~

通信至少是两个设备的事，需要相互兼容的硬件和软件支持，我们称之为通信协议。以太网通信在结构比较复杂，国际标准组织将整个以太网通信结构制定了OSI模型，
总共分层七个层，分别为应用层、表示层、会话层、传输层、网络层、数据链路层以及物理层，每个层功能不同，通信中各司其职，整个模型包括硬件和软件定义。
OSI模型是理想分层，一般的网络系统只是涉及其中几层。

TCP/IP是互联网最基本的协议，是互联网通信使用的网络协议，由网络层的IP协议和传输层的TCP协议组成。TCP/IP只有四个分层，
分别为应用层、传输层、网络层以及网络访问层。虽然TCP/IP分层少了，但与OSI模型是不冲突的，它把OSI模型一些层次整合一起的，本质上可以实现相同功能。

实际上，还有一个TCP/IP混合模型，分为五个层，参考图 TCP_IP混合参考模型_ ，它实际与TCP/IP四层模型是相通的，只是把网络访问层拆成数据链路层和物理层。
这种分层方法对我们学习理解更容易。

.. image:: media/ETH001.png
    :align: center
    :name: TCP_IP混合参考模型
    :alt: TCP/IP混合参考模型

设计网络时，为了降低网络设计的复杂性，对组成网络的硬件、软件进行封装、分层，这些分层即构成了网络体系模型。在两个设备相同层之间的对话、
通信约定，构成了层级协议。设备中使用的所有协议加起来统称协议栈。在这个网络模型中，每一层完成不同的任务，都提供接口供上一层访问。
而在每层的内部，可以使用不同的方式来实现接口，因而内部的改变不会影响其它层。

在TCP/IP混合参考模型中，数据链路层又被分为LLC层(逻辑链路层)和MAC层(媒体介质访问层)。目前，对于普通的接入网络终端的设备，
LLC层和MAC层是软、硬件的分界线。如PC的网卡主要负责实现参考模型中的MAC子层和物理层，在PC的软件系统中则有一套庞大程序实现了LLC层及以上的所有网络层次的协议。

由硬件实现的物理层和MAC子层在不同的网络形式有很大的区别，如以太网和Wi-Fi，这是由物理传输方式决定的。
但由软件实现的其它网络层次通常不会有太大区别，在PC上也许能实现完整的功能，一般支持所有协议，而在嵌入式领域则按需要进行裁剪。

以太网
~~~~~~~~~

以太网(Ethernet)是互联网技术的一种，由于它是在组网技术中占的比例最高，很多人直接把以太网理解为互联网。

以太网是指遵守IEEE 802.3标准组成的局域网，由IEEE802.3标准规定的主要是位于参考模型的物理层(PHY)和数据链路层中的介质访问控制子层(MAC)。
在家庭、企业和学校所组建的PC局域网形式一般也是以太网，其标志是使用水晶头网线来连接(当然还有其它形式)。IEEE还有其它局域网标准，
如IEEE 802.11是无线局域网，俗称Wi-Fi。IEEE802.15是个人域网，即蓝牙技术，其中的802.15.4标准则是ZigBee技术。

现阶段，工业控制、环境监测、智能家居的嵌入式设备产生了接入互联网的需求，利用以太网技术，嵌入式设备可以非常容易地接入到现有的计算机网络中。

PHY层
^^^^^^^^^^^^

在物理层，由IEEE 802.3标准规定了以太网使用的传输介质、传输速度、数据编码方式和冲突检测机制，物理层一般是通过一个PHY芯片实现其功能的。

传输介质
''''''''''''

传输介质包括\ *同轴电缆、双绞线(水晶头网线是一种双绞线)、光纤*\ 。根据不同的传输速度和距离要求，
基于这三类介质的信号线又衍生出很多不同的种类。最常用的是“五类线”适用于100BASE-T和10BASE-T的网络，它们的网络速率分别为100Mbps和10Mbps。

编码
''''''

为了让接收方在没有外部时钟参考的情况也能确定每一位的起始、结束和中间位置，在传输信号时不直接采用二进制编码。
在10BASE-T的传输方式中采用\ *曼彻斯特编码*\ ，在100BASE-T中则采用\ *4B/5B编码*\ 。

*曼彻斯特编码*\ 把每一个二进制位的周期分为两个间隔，在表示“1”时，以前半个周期为高电平，后半个周期为低电平。表示“0”时则相反，见图 曼彻斯特编码_

.. image:: media/ETH002.png
    :align: center
    :name: 曼彻斯特编码
    :alt: 曼彻斯特编码

采用曼彻斯特码在每个位周期都有电压变化，便于同步。但这样的编码方式效率太低，只有50%。

在100BASE-T 采用的4B/5B编码是把待发送数据位流的每4位分为一组，以特定的5位编码来表示，这些特定的5位编码能使数据流有足够多的跳变，
达到同步的目的，而且效率也从曼彻斯特编码的50%提高到了80%。

CSMA/CD冲突检测
'''''''''''''''''''''''''''''''

早期的以太网大多是多个节点连接到同一条网络总线上(总线型网络)，存在信道竞争问题，因而每个连接到以太网上的节点都必须具备冲突检测功能。
以太网具备CSMA/CD冲突检测机制，如果多个节点同时利用同一条总线发送数据，则会产生冲突，总线上的节点可通过接收到的信号与原始发送的信号的比较检测是否存在冲突，
若存在冲突则停止发送数据，随机等待一段时间再重传。

现在大多数局域网组建的时候很少采用总线型网络，大多是一个设备接入到一个独立的路由或交换机接口，组成星型网络，不会产生冲突。但为了兼容，新出的产品还是带有冲突检测机制。


MAC子层
~~~~~~~~~


MAC的功能
^^^^^^^^^^^^^

MAC子层是属于数据链路层的下半部分，它主要负责与物理层进行数据交接，如是否可以发送数据，发送的数据是否正确，
对数据流进行控制等。它自动对来自上层的数据包加上一些控制信号，交给物理层。接收方得到正常数据时，自动去除MAC控制信号，把该数据包交给上层。

MAC数据包
''''''''''''''''''

IEEE对以太网上传输的数据包格式也进行了统一规定，见图 MAC数据包格式_ 。该数据包被称为MAC数据包。

.. image:: media/ETH003.png
    :align: center
    :name: MAC数据包格式
    :alt: MAC数据包格式


MAC数据包由\ *前导字段、帧起始定界符、目标地址、源地址、数据包类型、数据域、填充域、校验和域*\ 组成。

-  前导字段，也称报头，这是一段方波，用于使收发节点的时钟同步。内容为连续7个字节的0x55。字段和帧起始定界符在MAC收到数据包后会自动过滤掉。

-  帧起始定界符(SFD)：用于区分前导段与数据段的，内容为0xD5。

-  MAC地址： MAC地址由48位数字组成，它是网卡的物理地址，在以太网传输的最底层，就是根据MAC地址来收发数据的。部分MAC地址用于广播和多播，
   在同一个网络里不能有两个相同的MAC地址。PC的网卡在出厂时已经设置好了MAC地址，但也可以通过一些软件来进行修改，
   在嵌入式的以太网控制器中可由程序进行配置。数据包中的DA是目标地址，SA是源地址。

-  数据包类型：本区域可以用来描述本MAC数据包是属于TCP/IP协议层的IP包、ARP包还是SNMP包，也可以用来描述本MAC数据包数据段的长度。
   如果该值被设置大于0x0600，不用于长度描述，而是用于类型描述功能，表示与以太网帧相关的MAC客户端协议的种类。

-  数据段：数据段是MAC包的核心内容，它包含的数据来自MAC的上层。其长度可以从0~1500字节间变化。

-  填充域：由于协议要求整个MAC数据包的长度至少为64字节(接收到的数据包如果少于64字节会被认为发生冲突，数据包被自动丢弃)，
   当数据段的字节少于46字节时，在填充域会自动填上无效数据，以使数据包符合长度要求。

-  校验和域：MAC数据包的尾部是校验和域，它保存了CRC校验序列，用于检错。

以上是标准的MAC数据包，IEEE 802.3同时还规定了扩展的MAC数据包，它是在标准的MAC数据包的SA和数据包类型之间添加4个字节的QTag前缀字段，
用于获取标志的MAC帧。前2个字节固定为0x8100，用于识别QTag前缀的存在；后两个字节内容分别为3个位的用户优先级、1个位的标准格式指示符(CFI)和一个12位的VLAN标识符。

TCP/IP协议栈
~~~~~~~~~~~~~~~~~~~~~~~~~

标准TCP/IP协议是用于计算机通信的一组协议，通常称为TCP/IP协议栈，通俗讲就是符合以太网通信要求的代码集合，
一般要求它可以实现图 TCP_IP混合参考模型_ 中每个层对应的协议，比如应用层的HTTP、FTP、DNS、SMTP协议，
传输层的TCP、UDP协议、网络层的IP、ICMP协议等等。关于TCP/IP协议详细内容推荐阅读《TCP-IP详解》和《用TCP/IP进行网际互连》理解。

Windows操作系统、UNIX类操作系统都有自己的一套方法来实现TCP/IP通信协议，它们都提供非常完整的TCP/IP协议。对于一般的嵌入式设备，
受制于硬件条件没办法支持使用在Window或UNIX类操作系统的运行的TCP/IP协议栈，一般只能使用简化版本的TCP/IP协议栈，
目前开源的适合嵌入式的有uIP、TinyTCP、uC/TCP-IP、LwIP等等。其中LwIP是目前在嵌入式网络领域被讨论和使用广泛的协议栈。本章内容其中一个目的就是移植LwIP到开发板上运行。

为什么需要协议栈
^^^^^^^^^^^^^^^^^^^^^^^^

物理层主要定义物理介质性质，MAC子层负责与物理层进行数据交接，这两部分是与硬件紧密联系的，就嵌入式控制芯片来说，很多都内部集成了MAC控制器，
完成MAC子层功能，所以依靠这部分功能是可以实现两个设备数据交换，而时间传输的数据就是MAC数据包，发送端封装好数据包，接收端则解封数据包得到可用数据，
这样的一个模型与使用USART控制器实现数据传输是非常类似的。但如果将以太网运用在如此基础的功能上，完全是大材小用，因为以太网具有传输速度快、
可传输距离远、支持星型拓扑设备连接等等强大功能。功能强大的东西一般都会用高级的应用，这也是设计者的初衷。

使用以太网接口的目的就是为了方便与其它设备互联，如果所有设备都约定使用一种互联方式，在软件上加一些层次来封装，这样不同系统、
不同的设备通讯就变得相对容易了。而且只要新加入的设备也使用同一种方式，就可以直接与之前存在于网络上的其它设备通讯。
这就是为什么产生了在MAC之上的\ *其它层次的网络协议*\及为什么要使用\ *协议栈*\ 的原因。又由于在各种协议栈中\ *TCP/IP协议栈*\ 得到了最广泛使用，
所有接入互联网的设备都遵守\ *TCP/IP协议*\ 。所以，想方便地与其它设备互联通信，需要提供对TCP/IP协议的支持。

各网络层的功能
^^^^^^^^^^^^^^^^^^^

用以太网和Wi-Fi作例子，它们的MAC子层和物理层有较大的区别，但在MAC之上的LLC层、网络层、传输层和应用层的协议，是基本相同的，
这几层协议由软件实现，并对各层进行封装。根据TCP/IP协议，各层的要实现的功能如下：

*LLC层：*\ 处理传输错误；调节数据流，协调收发数据双方速度，防止发送方发送得太快而接收方丢失数据。主要使用数据链路协议。

*网络层：*\ 本层也被称为IP层。LLC层负责把数据从线的一端传输到另一端，但很多时候不同的设备位于不同的网络中(并不是简单的网线的两头)。此时就需要网络层来解决子网路由拓扑问题、路径选择问题。在这一层主要有IP协议、ICMP协议。

*传输层：*\ 由网络层处理好了网络传输的路径问题后，端到端的路径就建立起来了。传输层就负责处理端到端的通讯。在这一层中主要有TCP、UDP协议

*应用层：*\ 经过前面三层的处理，通讯完全建立。应用层可以通过调用传输层的接口来编写特定的应用程序。而TCP/IP协议一般也会包含一些简单的应用程序如Telnet远程登录、FTP文件传输、SMTP邮件传输协议。

实际上，在发送数据时，经过网络协议栈的每一层，都会给来自上层的数据添加上一个数据包的头，再传递给下一层。在接收方收到数据时，
一层层地把所在层的数据包的头去掉，向上层递交数据，参考图 数据经过每一层的封装和还原_ 。

.. image:: media/ETH0035.png
    :align: center
    :name: 数据经过每一层的封装和还原
    :alt: 数据经过每一层的封装和还原

以太网外设(ETH)
~~~~~~~~~~~~~~~~~~~~~~~~~~

STM32H743系列控制器内部集成了一个以太网外设，它实际是一个通过DMA控制器进行介质访问控制(MAC)，它的功能就是实现MAC层的任务。
借助以太网外设，STM32H743控制器可以通过ETH外设按照IEEE 802.3-2002标准发送和接收MAC数据包。ETH内部自带专用的DMA控制器用于MAC，
ETH支持两个工业标准接口介质独立接口(MII)和简化介质独立接口(RMII)用于与外部PHY芯片连接。MII和RMII接口用于MAC数据包传输，
ETH还集成了站管理接口(SMI)接口专门用于与外部PHY通信，用于访问PHY芯片寄存器。

物理层定义了以太网使用的传输介质、传输速度、数据编码方式和冲突检测机制，PHY芯片是物理层功能实现的实体，生活中常用水晶头网线+水晶头插座+PHY组合构成了物理层。

ETH有专用的DMA控制器，它通过AHB主从接口与内核和存储器相连，AHB主接口用于控制数据传输，而AHB从接口用于访问“控制与状态寄存器”(CSR)空间。
在进行数据发送是，先将数据有存储器以DMA传输到发送TX FIFO进行缓冲，然后由MAC内核发送；接收数据时，RXFIFO先接收以太网数据帧，
再由DMA传输至存储器。ETH系统功能框图见图 ETH功能框图_ 。

.. image:: media/ETH004.png
    :align: center
    :name: ETH功能框图
    :alt: ETH功能框图


SMI接口
^^^^^^^^^^^^^

SMI是MAC内核访问PHY寄存器标志接口，它由两根线组成，数据线MDIO和时钟线MDC。SMI支持访问32个PHY，这在设备需要多个网口时非常有用，
不过一般设备都只使用一个PHY。PHY芯片内部一般都有32个16位的寄存器，用于配置PHY芯片属性、工作环境、状态指示等等，
当然很多PHY芯片并没有使用到所有寄存器位。MAC内核就是通过SMI向PHY的寄存器写入数据或从PHY寄存器读取PHY状态，
一次只能对一个PHY的其中一个寄存器进行访问。SMI最大通信频率为2.5MHz，通过控制以太网MAC MII地址寄存器 (ETH_MACMIIAR)的CR位可选择时钟频率。

SMI帧格式
''''''''''''''''''

SMI是通过数据帧方式与PHY通信的，帧格式如表 SMI帧格式_ ，数据位传输顺序从左到右。

.. image:: media/ETH01.png
    :align: center
    :name: SMI帧格式
    :alt: SMI帧格式

PADDR用于指定PHY地址，每个PHY都有一个地址，一般由PHY硬件设计决定，所以是固定不变的。RADDR用于指定PHY寄存器地址。
TA为状态转换域，若为读操作，MAC输出两个位高阻态，而PHY芯片则在第一位时输出高阻态，第二位时输出“0”。若为写操作，MAC输出“10”，PHY芯片则输出高阻态。
数据段有16位，对应PHY寄存器每个位，先发送或接收到的位对应以太网 MAC MII 数据寄存器(ETH_MACMIIDR)寄存器的位15。

SMI读写操作
'''''''''''''''''''

当以太网MAC MII地址寄存器 (ETH_MACMIIAR)的写入位和繁忙位被置1时，SMI将向指定的PHY芯片指定寄存器写入ETH_MACMIIDR中的数据。写操作时序见图 SMI写操作_ 。

.. image:: media/ETH005.png
    :align: center
    :name: SMI写操作
    :alt: SMI写操作


当以太网MAC MII地址寄存器 (ETH_MACMIIAR)的写入位为0并且繁忙位被置1时，SMI将从向指定的PHY芯片指定寄存器读取数据到ETH_MACMIIDR内。读操作时序见图 SMI读操作_ 。

.. image:: media/ETH006.png
    :align: center
    :name: SMI读操作
    :alt: SMI读操作


MII和RMII接口
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

介质独立接口(MII)用于连接MAC控制器和PHY芯片，提供数据传输路径。RMII接口是MII接口的简化版本，MII需要16根通信线，RMII只需7根通信，
在功能上是相同的。图 MII接口连接_ 为MII接口连接示意图，图 RMII接口连接_ 为RMII接口连接示意图。

.. image:: media/ETH007.png
    :align: center
    :name: MII接口连接
    :alt: MII接口连接


.. image:: media/ETH008.png
    :align: center
    :name: RMII接口连接
    :alt: RMII接口连接


-  TX_CLK：数据发送时钟线。标称速率为10Mbit/s时为2.5MHz；速率为100Mbit/s时为25MHz。RMII接口没有该线。

-  RX_CLK：数据接收时钟线。标称速率为10Mbit/s时为2.5MHz；速率为100Mbit/s时为25MHz。RMII接口没有该线。

-  TX_EN：数据发送使能。在整个数据发送过程保存有效电平。

-  TXD[3:0]或TXD[1:0]：数据发送数据线。对于MII有4位，RMII只有2位。只有在TX_EN处于有效电平数据线才有效。

-  CRS：载波侦听信号，由PHY芯片负责驱动，当发送或接收介质处于非空闲状态时使能该信号。在全双工模式该信号线无效。

-  COL：冲突检测信号，由PHY芯片负责驱动，检测到介质上存在冲突后该线被使能，并且保持至冲突解除。在全双工模式该信号线无效。

-  RXD[3:0]或RXD[1:0]：数据接收数据线，由PHY芯片负责驱动。对于MII有4位，RMII只有2位。在MII模式，当RX_DV禁止、RX_ER使能时，特定的RXD[3:0]值用于传输来自PHY的特定信息。

-  RX_DV：接收数据有效信号，功能类似TX_EN，只不过用于数据接收，由PHY芯片负责驱动。对于RMII接口，是把CRS和RX_DV整合成CRS_DV信号线，当介质处于不同状态时会自切换该信号状态。

-  RX_ER：接收错误信号线，由PHY驱动，向MAC控制器报告在帧某处检测到错误。

-  REF_CLK：仅用于RMII接口，由外部时钟源提供50MHz参考时钟。

因为要达到100Mbit/s传输速度，MII和RMII数据线数量不同，使用MII和RMII在时钟线的设计是完全不同的。对于MII接口，
一般是外部为PHY提供25MHz时钟源，再由PHY提供TX_CLK和RX_CLK时钟。对于RMII接口，一般需要外部直接提供50MHz时钟源，同时接入MAC和PHY。

开发板板载的PHY芯片型号为LAN8720A，该芯片只支持RMII接口，电路设计时参考图 RMII接口连接_ 。

ETH相关硬件在STM32H743控制器分布参考表 ETH复用引脚_ 。

.. image:: media/ETH02.png
    :align: center
    :name: ETH复用引脚
    :alt: ETH复用引脚

其中，PPS_OUT是IEEE 1588定义的一个时钟同步机制。

MAC数据包发送和接收
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ETH外设负责MAC数据包发送和接收。利用DMA从系统寄存器得到数据包数据内容，ETH外设自动填充完成MAC数据包封装，然后通过PHY发送出去。
在检测到有MAC数据包需要接收时，ETH外设控制数据接收，并解封MAC数据包得到解封后数据通过DMA传输到系统寄存器内。

MAC数据包发送
''''''''''''''''''''''''

MAC数据帧发送全部由DMA控制，从系统存储器读取的以太网帧由DMA推入FIFO，然后将帧弹出并传输到MAC内核。帧传输结束后，
从MAC内核获取发送状态并传回DMA。在检测到SOF(Start Of Frame)时，MAC接收数据并开始MII发送。在EOF(End Of Frame)传输到MAC内核后，
内核将完成正常的发送，然后将发送状态返回给DMA。如果在发送过程中发送常规冲突，MAC内核将使发送状态有效，然后接受并丢弃所有后续数据，
直至收到下一SOF。检测到来自MAC的重试请求时，应从SOF重新发送同一帧。如果发送期间未连续提供数据，MAC将发出下溢状态。在帧的正常传输期间，
如果MAC在未获得前一帧的EOF的情况下接收到SOF，则将忽略该SOF并将新的帧视为前一帧的延续。

MAC控制MAC数据包的发送操作，它会自动生成前导字段和SFD以及发送帧状态返回给DMA，在半双工模式下自动生成阻塞信号，
控制jabber(MAC看门狗)定时器用于在传输字节超过2048字节时切断数据包发送。在半双工模式下，MAC使用延迟机制进行流量控制，
程序通过将ETH_MACFCR寄存器的BPA位置1来请求流量控制。MAC包含符合IEEE 1588的时间戳快照逻辑。
MAC数据包发送时序参考图 MAC数据包发送时序_ 。

.. image:: media/ETH009.png
    :align: center
    :name: MAC数据包发送时序
    :alt: MAC数据包发送时序


MAC数据包接收
''''''''''''''''''''''''

MAC接收到的数据包填充RX FIFO，达到FIFO设定阈值后请求DMA传输。在默认直通模式下，当FIFO接收到64个字节(使用ETH_DMAOMR寄存器中的RTC位配置)或完整的数据包时，
数据将弹出，其可用性将通知给DMA。DMA向AHB接口发起传输后，数据传输将从FIFO持续进行，直到传输完整个数据包。完成EOF帧的传输后，
状态字将弹出并发送到DMA控制器。在Rx FIFO存储转发模式(通过ETH_DMAOMR寄存器中的RSF位配置)下，仅在帧完全写入Rx FIFO后才可读出帧。

当MAC在MII上检测到SFD时，将启动接收操作。MAC内核将去除报头和SFD，然后再继续处理帧。检查报头字段以进行过滤，
FCS字段用于验证帧的CRC如果帧未通过地址滤波器，则在内核中丢弃该帧。MAC数据包接收时序参考图 MAC数据包接收时序_ 。

.. image:: media/ETH010.png
    :align: center
    :name: MAC数据包接收时序
    :alt: MAC数据包接收时序


MAC过滤
^^^^^^^^^^^^^

MAC过滤功能可以选择性的过滤设定目标地址或源地址的MAC帧。它将检查所有接收到的数据帧的目标地址和源地址，根据过滤选择设定情况，
检测后报告过滤状态。针对目标地址过滤可以有三种，分别是单播、多播和广播目标地址过滤；针对源地址过滤就只有单播源地址过滤。

单播目标地址过滤是将接收的相应DA字段与预设的以太网MAC地址寄存器内容比较，最高可预设4个过滤MAC地址。
多播目标地址过滤是根据帧过滤寄存器中的HM位执行对多播地址的过滤，是对MAC地址寄存器进行比较来实现的。
单播和多播目标地址过滤都还支持Hash过滤模式。广播目标地址过滤通过将帧过滤寄存器的BFD位置1使能，这使得MAC丢弃所有广播帧。

单播源地址过滤是将接收的SA字段与SA寄存器内容进行比较过滤。

MAC过滤还具备反向过滤操作功能，即让过滤结构求补集。

PHY：LAN8720A
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

LAN8720A是SMSC公司(已被Microchip公司收购)设计的一个体积小、功耗低、全能型10/100Mbps的以太网物理层收发器。
它是针对消费类电子和企业应用而设计的。LAN8720A总共只有24Pin，仅支持RMII接口。由它组成的网络结构见图 由LAN8720A组成的网络系统结构_ 。

.. image:: media/ETH011.png
    :align: center
    :name: 由LAN8720A组成的网络系统结构
    :alt: 由LAN8720A组成的网络系统结构


LAN8720A通过RMII与MAC连接。RJ45是网络插座，在与LAN8720A连接之间还需要一个变压器，所以一般使用带电压转换和LED指示灯的HY911105A型号的插座。
一般来说，必须为使用RMII接口的PHY提供50MHz的时钟源输入到REF_CLK引脚，不过LAN8720A内部集成PLL，
可以将25MHz的时钟源陪频到50MHz并在指定引脚输出该时钟，所以我们可以直接使其与REF_CLK连接达到提供50MHz时钟的效果。

LAN8720A内部系统结构见图 LAN8720A内部系统结构_ 。

.. image:: media/ETH012.png
    :align: center
    :name: LAN8720A内部系统结构
    :alt: LAN8720A内部系统结构

LAN8720A有各个不同功能模块组成，最重要的要数接收控制器和发送控制器，其它的基本上都是与外部引脚挂钩，实现信号传输。部分引脚是具有双重功能的，
比如PHYAD0与RXER引脚是共用的，在系统上电后LAN8720A会马上读取这部分共用引脚的电平，以确定系统的状态并保存在相关寄存器内，之后则自动转入作为另一功能引脚。

PHYAD[0]引脚用于配置SMI通信的LAN8720A地址，在芯片内部该引脚已经自带下拉电阻，默认认为0(即使外部悬空不接)，在系统上电时会检测该引脚获取得到LAN8720A的地址为0或者1，
并保存在特殊模式寄存器(R18)的PHYAD位中，该寄存器的PHYAD有5个位，在需要超过2个LAN8720A时可以通过软件设置不同SMI通信地址。PHYAD[0]是与RXER引脚共用。

MODE[2:0]引脚用于选择LAN8720A网络通信速率和工作模式，可选10Mbps或100Mbps通信速度，半双工或全双工工作模式，另外LAN8720A支持HP Auto-MDIX自动翻转功能，
即可自动识别直连或交叉网线并自适应。一般将MODE引脚都设置为1，可以让LAN8720A启动自适应功能，它会自动寻找最优工作方式。MODE[0]与RXD0引脚共用、
MODE[1]与RXD1引脚共用、MODE[2]与CRS_DV引脚共用。

nINT/REFCLKO引脚用于RMII接口中REF_CLK信号线，当nINTSEL引脚为低电平是，它也可以被设置成50MHz时钟输出，
这样可以直接与STM32H743的REF_CLK引脚连接为其提供50MHz时钟源，这种模式要求为XTAL1与XTAL2之间或为XTAL1/CLKIN提供25MHz时钟，
由LAN8720A内部PLL电路倍频得到50MHz时钟，此时nIN/REFCLKO引脚的中断功能不可用，用于50MHz时钟输出。当nINTSEL引脚为高电平时，
LAN8720A被设置为时钟输入，即外部时钟源直接提供50MHz时钟接入STM32H743的REF_CLK引脚和LAN8720A的XTAL1/CLKIN引脚，
此时nINT/REFCLKO可用于中断功能。nINTSEL与LED2引脚共用，一般使用下拉

REGOFF引脚用于配置内部+1.2V电压源，LAN8720A内部需要+1.2V电压，可以通过VDDCR引脚输入+1.2V电压提供，也可以直接利用LAN8720A内部+1.2V稳压器提供。
当REGOFF引脚为低电平时选择内部+1.2V稳压器。REGOFF与LED1引脚共用。

SMI支持寻址32个寄存器，LAN8720A只用到其中14个，参考表 LAN8720A寄存器列表_ 。

.. image:: media/ETH03.png
    :align: center
    :name: LAN8720A寄存器列表
    :alt: LAN8720A寄存器列表

序号与SMI数据帧中的RADDR是对应的，这在编写驱动时非常重要，本文将它们标记为R0~R31。寄存器可规划为三个组：Basic、Extended和Vendor-specific。
Basic是IEEE 802.3要求的，R0是基本控制寄存器，其位15为SoftReset位，向该位写1启动LAN8720A软件复位，还包括速度、自适应、低功耗等等功能设置。
R1是基本状态寄存器。Extended是扩展寄存器，包括LAN8720A的ID号、制造商、版本号等等信息。Vendor-specific是供应商自定义寄存器，
R31是特殊控制/状态寄存器，指示速度类型和自适应功能。

LwIP：轻型TCP/IP协议栈
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

LwIP是Light Weight Internet Protocol 的缩写，是由瑞士计算机科学院AdamDunkels等开发的适用于嵌入式领域的开源轻量级TCP/IP协议栈。
它可以移植到含有操作系统的平台中，也可以在无操作系统的平台下运行。由于它开源、占用的RAM和ROM比较少、支持较为完整的TCP/IP协议、
且十分便于裁剪、调试，被广泛应用在中低端的32位控制器平台。可以访问网站：\
http://savannah.nongnu.org/projects/lwip/ 获取更多LwIP信息。

目前，LwIP最新更新到2.0.3版本，我们在上述网站可找到相应的LwIP源码下载通道。
我们下载两个压缩包：lwip-2.0.3.zip和contrib-2.01.zip，lwip-2.0.3.zip包括了LwIP的实现代码，
contrib-2.0.3.zip包含了不同平台移植LwIP的驱动代码和使用LwIP实现的一些应用实例测试。

但是，遗憾的是contrib-2.0.3.zip并没有为STM32平台提供实例，这对于初学者想要移植LwIP来说难度还是非常大的。
ST公司也是认识到LwIP在嵌入式领域的重要性，所以他们针对LwIP应用开发了测试平台。为减少移植工作量，
我们选择使用ST官方HAL开发包《STM32Cube_FW_H7_V1.5.0》中一个例程的相关接口文件，
工程路径为STM32Cube_FW_H7_V1.5.0\Projects\STM32H743I_EVAL\Applications\LwIP\LwIP_TCP_Echo_Client的，
这样我们也可以花更多精力在理解代码实现方法上。

本章的一个重点内容就是介绍LwIP移植至我们的开发平台，详细的操作步骤参考下文介绍。

ETH初始化结构体详解
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

从STM32的ETH外设我们了解到它的功能非常多，控制涉及的寄存器也非常丰富，而使用STM32 HAL库提供的各种结构体及库函数可以简化这些控制过程。
跟其它外设一样，STM32HAL库提供了初始化结构体成员用于设置ETH工作环境参数，并由ETH相应初始化配置函数或功能函数调用，
这些设定参数将会设置ETH相应的寄存器，达到配置ETH工作环境的目的。这些内容都定义在库文件“stm32h7xx_hal_eth.h”及“stm32h7xx_hal_eth.c”中，
编程时我们可以结合这两个文件内的注释使用或参考库帮助文档。



.. code-block:: c
    :caption: 代码清单:ETH-1 ETH_HandleTypeDef外设管理结构体（文件stm32h7xx_hal_eth.h）
    :name: 代码清单:ETH-1
    :linenos:

    typedef struct {
        ETH_TypeDef                *Instance; /*!< 外设寄存器基地址*/
        ETH_InitTypeDef            Init; /*!< 初始化结构体*/
        ETH_TxDescListTypeDef      TxDescList; /*!< 发送数据描述符*/
        ETH_RxDescListTypeDef      RxDescList; /*!< 接收数据描述符*/
        HAL_LockTypeDef            Lock; /*!< 锁资源*/
        __IO HAL_ETH_StateTypeDef  gState; /*!< ETH工作状态*/
        __IO HAL_ETH_StateTypeDef  RxState; /*!< ETH接收状态*/
        __IO uint32_t              ErrorCode; /*!< 错误操作返回值*/
        __IO uint32_t              DMAErrorCode; /*!< DMA错误执行操作返回值*/
        __IO uint32_t              MACErrorCode; /*!< MAC错误操作返回值*/
        __IO uint32_t              MACWakeUpEvent; /*!< MAC控制器唤醒事件*/
        __IO uint32_t              MACLPIEvent; /*!< MAC控制器低功耗空闲事件 */
    } ETH_HandleTypeDef;


(1)
Instance： 寄存器基地址指针，所有参数都是指定基地址后才能正确写入寄存器。

(2)
Init：ETH初始化结构体，下面会详细讲解每一个成员。

(3)
TxDescList：发送描述符，也就是发送缓冲区指针，指向当前发送数组的首地址。

(4)
RxDescList：接收描述符，接收缓冲区指针针，指向当前接收数组的首地址。

(5)
Lock：ETH外设锁资源。

(6)
gState：ETH的工作状态值以及发送过程的状态值，可以是HAL_ETH_STATE_RESET，HAL_ETH_STATE_READY等等。

(7)
RxState：ETH接收过程的状态值，其值与gState一样，也是HAL_ETH_StateTypeDef类型的枚举变量。

(8)
ErrorCode：ETH错误操作返回值，用户可以根据这个参数，找到代码的错误。

(9
DMAErrorCode：DMA错误操作返回值，用户可以根据这个参数，找到代码的错误。

(10)
MACErrorCode：MAC控制器错误操作返回值，用户可以根据这个参数，找到代码的错误。

(11)
MACWakeUpEvent：MAC控制器的唤醒事件来源。

(12)
MACLPIEvent：MAC控制器低功耗状态下的事件来源。

..

   有关ETH_HandleTypeDef外设管理结构体的成员中，我们只需要关注Instance和Init，这两个成员的变量。至于其他的变量，在调用HAL库的函数时，
   HAL库会根据外设的不同状态分别赋值，不需要用户关心。下面我们看一下，ETH的初始化结构体。

.. code-block:: c
    :caption: 代码清单:ETH-2 ETH_InitTypeDef（文件stm32h7xx_hal_eth.h）
    :name: 代码清单:ETH-2
    :linenos:

    typedef struct {
        uint8_t                     *MACAddr; /*!< MAC地址 */
        ETH_MediaInterfaceTypeDef   MediaInterface; /*!<选择总线类型*/
        ETH_DMADescTypeDef          *TxDesc;  /*!< 发送DMA描述符 */
        ETH_DMADescTypeDef          *RxDesc;  /*!< 接收DMA描述符 */
        uint32_t                    RxBuffLen;  /*!< 接收数组长度 */
    } ETH_InitTypeDef;


-  MACAddr：MAC地址指针，必须是一个6个元素数组的指针。

-  MediaInterface：以太网介质接口，可以是MII介质接口或者RMII介质接口。

-  TxDesc：发送DMA描述符列表，指向当前发送数组的首地址。

-  RxDesc：接收DMA描述符列表，指向当前接收数组的首地址。

-  RxBuffLen：接收数组的长度。


.. code-block:: c
    :caption: 代码清单:ETH-3ETH_MACConfigTypeDef（文件stm32h7xx_hal_eth.h）
    :name: 代码清单:ETH-3
    :linenos:

    typedef struct {
        uint32_t         SourceAddrControl;/*!< 源地址过滤 */
        FunctionalState  ChecksumOffload;  /*!<校验和减荷*/
        uint32_t         InterPacketGapVal;/*!< 控制发送帧间的最小间隙*/
        FunctionalState  GiantPacketSizeLimitControl;/*!< Giant数据包的大小限制 */
        FunctionalState  Support2KPacket;             /*!< 支持2K大小的数据包*/
        FunctionalState  CRCStripTypePacket;          /*!< 去除CRC字节*/
        FunctionalState  AutomaticPadCRCStrip;/*!<自动去除PAD和FCS字段*/
        FunctionalState  Watchdog;       /*!<以太网看门狗*/
        FunctionalState  Jabber;         /*!< jabber定时器功能*/
        FunctionalState  JumboPacket;    /*!< 是否接收载波值 */
        uint32_t         Speed;            /*!< 以太网速度选择*/
        uint32_t         DuplexMode;       /*!< 工作模式*/
        FunctionalState  LoopbackMode;     /*!< 回送模式*/
        FunctionalState  CarrierSenseBeforeTransmit;  /*!< 载波侦听 */
        FunctionalState  ReceiveOwn;/*!< 接收自身*/
        FunctionalState  CarrierSenseDuringTransmit; /*!< 传输过程中载波侦听*/
        FunctionalState  RetryTransmission;/*!< 传输重试*/
        uint32_t         BackOffLimit;    /*!<后退限制*/
        FunctionalState  DeferralCheck;   /*!< 检查延迟*/
        uint32_t         PreambleLength;  /*!< 前导码的长度*/
        FunctionalState  UnicastSlowProtocolPacketDetect;/*!<单播模式下慢速协议检测*/
        FunctionalState  SlowProtocolDetect;          /*!< 慢速协议检测 */
        FunctionalState  CRCCheckingRxPackets;        /*!< 接收数据的CRC检验 */
        uint32_t         GiantPacketSizeLimit;        /*!< Giant数据大小的阈值*/
        FunctionalState  ExtendedInterPacketGap;      /*!< 扩展IPG选择*/
        uint32_t         ExtendedInterPacketGapVal;   /*!< 扩展IPG*/
        FunctionalState  ProgrammableWatchdog;        /*!< 可编程看门狗*/
        uint32_t         WatchdogTimeout;             /*!< 超时检测*/
        uint32_t        PauseTime;                   /*!< 暂停时间*/
        FunctionalState  ZeroQuantaPause;             /*!< 零时间片暂停*/
        uint32_t         PauseLowThreshold;           /*!< 暂停阈值下限 */
        FunctionalState  TransmitFlowControl;         /*!< 发送流控制 */
        FunctionalState  UnicastPausePacketDetect;    /*!< 单播暂停帧检测*/
        FunctionalState  ReceiveFlowControl;          /*!< 接收流控制 */
        uint32_t         TransmitQueueMode;           /*!< 发送队列模式*/
        uint32_t         ReceiveQueueMode;            /*!< 接收队列模式*/
        FunctionalState  DropTCPIPChecksumErrorPacket; /*!< 丢弃TCPIP校验和错误数据*/
        FunctionalState  ForwardRxErrorPacket;        /*!< 转发错误帧 */
        FunctionalState  ForwardRxUndersizedGoodPacket;  /*!< 转发过小的好帧*/
    } ETH_MACConfigTypeDef;


-  SourceAddrControl：源地址过滤，可选源地址过滤、源地址反向过滤或禁用源地址过滤，它设定ETH_MACFFR寄存器SAF位和SAIF位的值。一般选择禁用源地址过滤。

-  ChecksumOffload：IPv4校验和减荷功能选择，可选使能或禁止，它设定ETH_MACCR寄存器IPCO位的值，
   当该位被置1时使能接收的帧有效载荷的TCP/UDP/ICMP标头的IPv4校验和检查。一般选择禁用，此时PCE和IP HCE状态位总是为0。

-  InterPacketGapVal：控制发送帧间的最小间隙，可选96bit时间、88bit时间、…、40bit时间，他设定ETH_MACCR寄存器的IFG[2:0]位的值，一般设置96bit时间。

-  GiantPacketSizeLimitControl：启用或禁用巨型数据包大小限制控制。

-  Support2KPacket： IEEE 802.3是否支持2K大小的数据包

-  CRCStripTypePacket：自动去除数据包的CRC字段功能，可选使能或禁用。

-  AutomaticPadCRCStrip：自动去除PAD和FCS字段功能，可选使能或禁用，它设定ETH_MACCR寄存器APCS位的值。当设置为1时，
   MAC在长度字段值小于或等于1500自己是去除传入帧上的PAD和FCS字段。一般禁止自动去除PAD和FCS字段功能。

-  Watchdog：以太网看门狗功能选择，可选使能或禁止，它设定以太网MAC配置寄存器(ETH_MACCR)的WD位的值。如果设置为1，
   使能看门狗，在接收MAC帧超过2048字节时自动切断后面数据，一般选择使能看门狗。如果设置为0，禁用看门狗，最长可接收16384字节的帧。

-  JumboPacket：是否接受载波值，可以选择使能或者是不使能。最多能接收9，018个字节。

-  Speed：以太网速度选择，可选10Mbps或100Mbps，它设定ETH_MACCR的FES位的值，一般设置100Mbps。但在使能自适应功能之后该位设置无效。

-  DuplexMode：以太网工作模式选择，可选全双工模式或半双工模式，它设定ETH_MACCR寄存器DM位的值。一般选择全双工模式，在使能了自适应功能后该成员设置无效。

-  LoopbackMode：回送模式选择，可选使能或禁止，它设定ETH_MACCR寄存器的LM位的值，当设置为1时，使能MAC在MII回送模式下工作。

-  CarrierSenseBeforeTransmit：载波侦听功能选择，可选使能或禁止，它设定ETH_MACCR寄存器的CSD位的值。
   当被设置为低电平时，MAC发送器会生成载波侦听错误，一般使能载波侦听功能。

-  ReceiveOwn：接收自身帧功能选择，可选使能或禁止，它设定ETH_MACCR寄存器的ROD位的值，当设置为0时，
   MAC接收发送时PHY提供的所有MAC包，如果设置为1，MAC禁止在半双工模式下接收帧。一般使能接收。

-  CarrierSenseDuringTransmit：传输过程中载波侦听功能选择，可选使能或禁止，它设定ETH_MACCR寄存器的CSD位的值。
   当被设置为低电平时，MAC发送器会生成载波侦听错误，一般使能载波侦听功能。

-  RetryTransmission：传输重试功能，可选使能或禁止，它设定ETH_MACCR寄存器RD位的值，当被设置为1时，MAC仅尝试发送一次，
   设置为0时，MAC会尝试根据BL的设置进行重试。一般选择使能重试。

-  BackOffLimit：后退限制，在发送冲突后重新安排发送的延迟时间，可选10、8、4、1，它设定ETH_MACCR寄存器BL位的值。一般设置为10。

-  DeferralCheck：检查延迟，可选使能或禁止，它设定ETH_MACCR寄存器DC位的值，当设置为0时，禁止延迟检查功能，MAC发送延迟，直到CRS信号变成无效信号。

-  PreambleLength：选择或不选择传输数据包的前导码长度（全双工模式）。

-  UnicastSlowProtocolPacketDetect：使用单播地址启用或禁用“慢速协议数据包检测”。

-  SlowProtocolDetect：启用或禁用慢速协议检测。

-  CRCCheckingRxPackets：启用或禁用接收数据包的CRC校验。

-  GiantPacketSizeLimit：指定MAC将其声明为Giant的数据包大小，此参数的值必须是1518字节和32KB之间。

-  ExtendedInterPacketGap：启用或禁用扩展的数据包间隙。

-  ExtendedInterPacketGapVal：在传输过程中设置数据包之间的扩展IPG，此参数可以是0x0到0xFF之间的值

-  ProgrammableWatchdog：启用或禁用可编程看门狗。

-  WatchdogTimeout：指定接收数据包的监视程序超时

-  PauseTime：暂停时间，保留发送控制帧中暂停时间字段要使用的值，可设置0至65535，它设定以太网MAC流控制寄存器(ETH_MACFCR)PT位的值。

-  ZeroQuantaPause：零时间片暂停，可选使用或禁止，它设定ETH_MACFCR寄存器ZQPD位的值。当设置为1时，当来自FIFO层的流控制信号去断言后，
   此位会禁止自动生成零时间片暂停控制帧。一般选择禁止。

-  PauseLowThreshold：暂停阈值下限，配置暂停定时器的阈值，达到该值值时，会自动程序传输暂停帧，可选暂停时间减去4个间隙、
   28个间隙、144个间隙或256个间隙，它设定ETH_MACFCR寄存器PLT位的值。一般选择暂停时间减去4个间隙。

-  TransmitFlowControl：发送流控制，可选使能或禁止，它设定ETH_MACFCR寄存器TFCE位的值。在全双工模式下，当设置为1时，
   MAC将使能流控制操作来发送暂停帧；为0时，将禁止MAC中的流控制操作，MAC不会传送任何暂停帧。在半双工模式下，当设置为1时，
   MAC将使能背压操作；为0时，将禁止背压功能。

-  UnicastPauseFrameDetect：单播暂停帧检测，可选使能或禁止，它设定ETH_MACFCR寄存器UPFD位的值。当设置为1时，
   MAC除了检测具有唯一多播地址的暂停帧外，还会检测具有ETH_MACA0HR和ETH_MACA0LR寄存器所指定的站单播地址的暂停帧。一般设置为禁止。

-  ReceiveFlowControl：接收流控制，可选使能或禁止，它设定ETH_MACFCR寄存器RFCE位的值。当设定为1时，MAC对接收到的暂停帧进行解码，
   并禁止其在指定时间（暂停时间）内发送；当设置为0时，将禁止暂停帧的解码功能，一般设置为禁止。

-  TransmitQueueMode：指定传输队列操作模式。

-  ReceiveQueueMode：指定接收队列操作模式。

-  DropTCPIPChecksumErrorPacket：丢弃TCP/IP校验错误帧，可选使能或禁止，它设定以太网DMA工作模式寄存器（ETH_DMAOMR）DTCEFD位的值，
   当设置为1时，如果帧中仅存在由接受校验和减荷引擎检测出来的错误，则内核不会丢弃它；为0时，如果FEF位复位，则会丢弃所有错误帧。

-  ForwardRxErrorPacket：转发错误帧，可选使能或禁止，它设定ETH_DMAOMR寄存器FEF位的值，当设置为1时，除了段错误帧之外所有帧都会转发到 DMA；
   为0时，RXFIFO会丢弃所有错误状态的帧。一般选择禁止。

-  ForwardRxUndersizedGoodPacket：转发过小的好帧，可选使能或禁止。它设定ETH_DMAOMR寄存器FUGF位的值，当设置为1时，
   RXFIFO会转发包括PAD和FCS字段的过小帧；为0时，会丢弃小于64字节的帧，除非接收阈值被设置为更低。

以太网通信实验：无操作系统LwIP移植
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

LwIP可以在带操作系统上运行，亦可在无操作系统上运行，这一实验我们讲解在无操作系统的移植步骤，并实现简单的传输代码，
后续章节会讲解在带操作系统移植过程，一般都是在无操作系统基础上修改而来的。

硬件设计
^^^^^^^^^^^^

在讲解移植步骤之前，有必须先介绍我们的实验硬件设计，主要是LAN8720A通过RMII和SMI接口与STM32H743x控制器连接，见图 PHY硬件设计_ 。

.. image:: media/ETH013.png
    :align: center
    :name: PHY硬件设计
    :alt: PHY硬件设计


电路设计时，将NINTSEL引脚通过下拉电阻拉低，设置NINT/FEFCLKO为输出50MHz时钟，当然前提是在XTAL1和XTAL2接入了25MHz的时钟源。
另外也把REGOFF引脚通过下拉电阻拉低，使能使用内部+1.2V稳压器。

移植步骤
^^^^^^^^^^^^

之前已经介绍了LwIP源代码(lwip-2.0.3.zip)和ST官方LwIP测试平台资料(STM32Cube_FW_H7_V1.5.0)下载，
由于ST官方提供的LwIP的最新版本为2.0.0，所以我们移植步骤是基于这两份资料进行的。

下面介绍无操作系统移植LwIP需要的文件。lwip-2.0.3.zip文件解压后参考图 LwIP官方下载文件解压目录_ 。
STM32Cube_FW_H7_V1.5.0文件解压后在Middlewares文件夹下的目录参考图 LwIP在CUBE开发包中作为中间件的文件目录_ 。
我们将STM32Cube_FW_H7_V1.5.0\Middlewares\Third_Party\LwIP\system路径下的system文件夹拷贝到lwip-2.0.3文件夹下。
两者结合得到我们最终需要移植的文件目录。新的lwip-2.0.3的文件目录参考图 LwIP最终需要移植的文件目录_ 。

.. image:: media/ETH014.png
    :align: center
    :name: LwIP官方下载文件解压目录
    :alt: LwIP官方下载文件解压目录


.. image:: media/ETH015.png
    :align: center
    :name: LwIP在CUBE开发包中作为中间件的文件目录
    :alt: LwIP在CUBE开发包中作为中间件的文件目录


.. image:: media/ETH016.png
    :align: center
    :name: LwIP最终需要移植的文件目录
    :alt: LwIP最终需要移植的文件目录


我们先来熟悉LwIP栈的目录组织，如图 LwIP最终需要移植的文件目录_ 所示，其中，doc包含文档文件；src包含LwIP栈的源代码文件；
api包含Netconn和套接字API文件；apps包含LwIP一些应用文件；core包含LwIP内核文件；include包含LwIP头文件；netif包含网络接口文件；
system包含LwIP端口硬件实现文件；arch包含STM32架构端口文件（所用的数据类型）OS包含使用操作系统的LwIP端口实现文件；
test包含LwIP官方的一些测试示例。

接下来，我们就根据图中文件结构详解移植过程。实验例程有需要用到系统滴答定时器systick、调试串口USART、LED灯功能，
对这些功能实现不做具体介绍，可以参考相关章节理解。

第一步：相关文件拷贝到工程目录


首先，上面已经准备好协议栈文件，我们将其拷贝到工程目录的USER文件夹下，并新建APP，BSP两个文件夹，BSP文件夹下放LED，UART，
LAN8720A等板载外设驱动，最终的文件结构见图 LwIP相关文件拷贝_ ，arch存放与开发平台相关头文件。

.. image:: media/ETH017.png
    :align: center
    :name: LwIP相关文件拷贝
    :alt: LwIP相关文件拷贝


lwip-2.0.3文件夹下的doc文件夹存放LwIP版权、移植、使用等等说明文件，移植之前有必须认真浏览一遍；src文件夹存放LwIP的实现代码，
也是我们工程代码真正需要的文件；test文件夹存放LwIP部分功能测试例程；另外，还有一些无后缀名的文件，都是一些说明性文件，可用记事本直接打开浏览。
port文件夹存放LwIP与STM32平台连接的相关文件，正如上面所说contrib-1.4.1.zip包含了不同平台移植代码，不过遗憾地是没有STM32平台的，
所以我们需要从ST官方提供的测试平台找到这部分连接代码，也就是port文件夹的内容。

接下来，在BSP文件下新建一个LAN8720A文件夹，用于存放以太网PHY相关驱动文件，包括两个部分文件，LAN8720A.h和LAN8720A.c，
这两个文件包含相关GPIO初始化，直接硬件相关的文件，如果硬件有更改只需要改这两个文件。

在APP文件夹下，我们参考ST官方LwIP测试平台的一个例程，
在如下目录STM32Cube_FW_H7_V1.5.0\Projects\STM32H743I_EVAL\Applications\LwIP\LwIP_TCP_Echo_Client中的Src文件夹和Inc文件夹中，
这里我们需要用到五个文件lwipopts.h、app_ethernet.h、app_ethernet.c、ethernetif.h、ethernetif.c，
因为例程使用的PHY型号不是使用LAN8720A，所以这四个文件需要我们进行修改。

第二部：为工程添加文件


第一步已经把相关的文件拷贝到对应的文件夹中，接下来就可以把需要用到的文件添加到工程中。图 LwIP官方下载文件解压目录_ 已经指示出来工程需要用到的*.c文件，
所以最终工程文件结构见图 工程文件结构_ ，图中api、ipv4和core都包含了对应文件夹下的所有*.c文件。Netif文件夹下只需要添加ethernet.c文件。

.. image:: media/ETH018.png
    :align: center
    :name: 工程文件结构
    :alt: 工程文件结构


接下来，还需要在工程选择中添加相关头文件路径，参考图 添加相关头文件路径_ 。

.. image:: media/ETH019.png
    :align: center
    :name: 添加相关头文件路径
    :alt: 添加相关头文件路径


第三步：文件修改


ethernetif.c文件是无操作系统时网络接口函数，该文件在移植时需要根据实际硬件初始化网络相关IO口，以及需要指定的内存空间作为缓存。
该文件主要有三个部分函数，HAL_ETH_MspInit函数用于初始化系统硬件接口；low_level_init函数用于初始化MAC相关工作环境、
初始化DMA描述符链表，并使能MAC和DMA； low_level_output函数是最底层发送一帧数据函数；low_level_input函数是最底层接收一帧数据函数。
sys_now函数获取当前时间的一个函数；ethernetif_init函数初始化网络接口结构 （netif）并调用 low_level_init 以初始化以太网外设；
ethernet_input函数调用 low_level_input接收包，然后将其提供给 LwIP 栈。

app_ethernet.c文件主要是实际的网络初始化应用程序，这里包含两个函数，Netif_Config函数是创建一个网络接口；User_notification函数是指示当前网络连接的状态。

LAN8720A.h和LAN8720A.c两个文件是ETH外设相关的底层配置，主要是 GPIO初始化即相关时钟使能。



.. code-block:: c
    :caption: 代码清单:ETH-4 ETH_GPIO_Config函数（文件LAN8720a.c）
    :name: 代码清单:ETH-4
    :linenos:

    /* ETH_MDIO */
    #define ETH_MDIO_GPIO_CLK_ENABLE()          __GPIOA_CLK_ENABLE()
    #define ETH_MDIO_PORT                       GPIOA
    #define ETH_MDIO_PIN                        GPIO_PIN_2
    #define ETH_MDIO_AF                         GPIO_AF11_ETH

    /* ETH_MDC */
    #define ETH_MDC_GPIO_CLK_ENABLE()           __GPIOC_CLK_ENABLE();
    #define ETH_MDC_PORT                        GPIOC
    #define ETH_MDC_PIN                         GPIO_PIN_1
    #define ETH_MDC_AF                          GPIO_AF11_ETH

    /* ETH_RMII_REF_CLK */
    #define ETH_RMII_REF_CLK_GPIO_CLK_ENABLE()  __GPIOA_CLK_ENABLE();
    #define ETH_RMII_REF_CLK_PORT               GPIOA
    #define ETH_RMII_REF_CLK_PIN                GPIO_PIN_1
    #define ETH_RMII_REF_CLK_AF                 GPIO_AF11_ETH

    /* ETH_RMII_CRS_DV */
    #define ETH_RMII_CRS_DV_GPIO_CLK_ENABLE()   __GPIOA_CLK_ENABLE();
    #define ETH_RMII_CRS_DV_PORT                GPIOA
    #define ETH_RMII_CRS_DV_PIN                 GPIO_PIN_7
    #define ETH_RMII_CRS_DV_AF                  GPIO_AF11_ETH

    /* ETH_RMII_RXD0 */
    #define ETH_RMII_RXD0_GPIO_CLK_ENABLE()     __GPIOC_CLK_ENABLE();
    #define ETH_RMII_RXD0_PORT                  GPIOC
    #define ETH_RMII_RXD0_PIN                   GPIO_PIN_4
    #define ETH_RMII_RXD0_AF                    GPIO_AF11_ETH

    /* ETH_RMII_RXD1 */
    #define ETH_RMII_RXD1_GPIO_CLK_ENABLE()     __GPIOC_CLK_ENABLE();
    #define ETH_RMII_RXD1_PORT                  GPIOC
    #define ETH_RMII_RXD1_PIN                   GPIO_PIN_5
    #define ETH_RMII_RXD1_AF                    GPIO_AF11_ETH

    /* ETH_RMII_TX_EN */
    #define ETH_RMII_TX_EN_GPIO_CLK_ENABLE()    __GPIOB_CLK_ENABLE();
    #define ETH_RMII_TX_EN_PORT                 GPIOB
    #define ETH_RMII_TX_EN_PIN                  GPIO_PIN_11
    #define ETH_RMII_TX_EN_AF                   GPIO_AF11_ETH

    /* ETH_RMII_TXD0 */
    #define ETH_RMII_TXD0_GPIO_CLK_ENABLE()     __GPIOG_CLK_ENABLE();
    #define ETH_RMII_TXD0_PORT                  GPIOG
    #define ETH_RMII_TXD0_PIN                   GPIO_PIN_13
    #define ETH_RMII_TXD0_AF                    GPIO_AF11_ETH

    /* ETH_RMII_TXD1 */
    #define ETH_RMII_TXD1_GPIO_CLK_ENABLE()     __GPIOG_CLK_ENABLE();
    #define ETH_RMII_TXD1_PORT                  GPIOG
    #define ETH_RMII_TXD1_PIN                   GPIO_PIN_14
    #define ETH_RMII_TXD1_AF                    GPIO_AF11_ETH
    /**
    * @brief  配置以太网接口
    * @param  None
    * @retval None
    */
    void ETH_GPIO_Config(void)
    {
        GPIO_InitTypeDef GPIO_InitStructure;
        /* 使能端口时钟 */
        ETH_MDIO_GPIO_CLK_ENABLE();
        ETH_MDC_GPIO_CLK_ENABLE();
        ETH_RMII_REF_CLK_GPIO_CLK_ENABLE();
        ETH_RMII_CRS_DV_GPIO_CLK_ENABLE();
        ETH_RMII_RXD0_GPIO_CLK_ENABLE();
        ETH_RMII_RXD1_GPIO_CLK_ENABLE();
        ETH_RMII_TX_EN_GPIO_CLK_ENABLE();
        ETH_RMII_TXD0_GPIO_CLK_ENABLE();
        ETH_RMII_TXD1_GPIO_CLK_ENABLE();

        /* 配置以太网引脚*/
        /*
        ETH_MDIO -------------------------> PA2
        ETH_MDC --------------------------> PC1
        ETH_MII_RX_CLK/ETH_RMII_REF_CLK---> PA1
        ETH_MII_RX_DV/ETH_RMII_CRS_DV ----> PA7
        ETH_MII_RXD0/ETH_RMII_RXD0 -------> PC4
        ETH_MII_RXD1/ETH_RMII_RXD1 -------> PC5
        ETH_MII_TX_EN/ETH_RMII_TX_EN -----> PB11
        ETH_MII_TXD0/ETH_RMII_TXD0 -------> PG13
        ETH_MII_TXD1/ETH_RMII_TXD1 -------> PG14
        */

        /* 配置ETH_MDIO引脚 */
        GPIO_InitStructure.Pin = ETH_MDIO_PIN;
        GPIO_InitStructure.Speed = GPIO_SPEED_HIGH;
        GPIO_InitStructure.Mode = GPIO_MODE_AF_PP;
        GPIO_InitStructure.Pull = GPIO_NOPULL;
        GPIO_InitStructure.Alternate = ETH_MDIO_AF;
        HAL_GPIO_Init(ETH_MDIO_PORT, &GPIO_InitStructure);

        /* 配置ETH_MDC引脚 */
        GPIO_InitStructure.Pin = ETH_MDC_PIN;
        GPIO_InitStructure.Alternate = ETH_MDC_AF;
        HAL_GPIO_Init(ETH_MDC_PORT, &GPIO_InitStructure);

        /* 配置ETH_RMII_REF_CLK引脚 */
        GPIO_InitStructure.Pin = ETH_RMII_REF_CLK_PIN;
        GPIO_InitStructure.Alternate = ETH_RMII_REF_CLK_AF;
        HAL_GPIO_Init(ETH_RMII_REF_CLK_PORT, &GPIO_InitStructure);

        /* 配置ETH_RMII_CRS_DV引脚 */
        GPIO_InitStructure.Pin = ETH_RMII_CRS_DV_PIN;
        GPIO_InitStructure.Alternate = ETH_RMII_CRS_DV_AF;
        HAL_GPIO_Init(ETH_RMII_CRS_DV_PORT, &GPIO_InitStructure);

        /* 配置ETH_RMII_RXD0引脚 */
        GPIO_InitStructure.Pin = ETH_RMII_RXD0_PIN;
        GPIO_InitStructure.Alternate = ETH_RMII_RXD0_AF;
        HAL_GPIO_Init(ETH_RMII_RXD0_PORT, &GPIO_InitStructure);

        /* 配置ETH_RMII_RXD1引脚 */
        GPIO_InitStructure.Pin = ETH_RMII_RXD1_PIN;
        GPIO_InitStructure.Alternate = ETH_RMII_RXD1_AF;
        HAL_GPIO_Init(ETH_RMII_RXD1_PORT, &GPIO_InitStructure);

        /* 配置ETH_RMII_TX_EN引脚 */
        GPIO_InitStructure.Pin = ETH_RMII_TX_EN_PIN;
        GPIO_InitStructure.Alternate = ETH_RMII_TX_EN_AF;
        HAL_GPIO_Init(ETH_RMII_TX_EN_PORT, &GPIO_InitStructure);

        /* 配置ETH_RMII_TXD0引脚 */
        GPIO_InitStructure.Pin = ETH_RMII_TXD0_PIN;
        GPIO_InitStructure.Alternate = ETH_RMII_TXD0_AF;
        HAL_GPIO_Init(ETH_RMII_TXD0_PORT, &GPIO_InitStructure);

        /* 配置ETH_RMII_TXD1引脚 */
        GPIO_InitStructure.Pin = ETH_RMII_TXD1_PIN;
        GPIO_InitStructure.Alternate = ETH_RMII_TXD1_AF;
        HAL_GPIO_Init(ETH_RMII_TXD1_PORT, &GPIO_InitStructure);
    }


HAL_ETH_MspInit函数调用ETH_GPIO_Config进行硬件初始化，并使能以太网时钟。

.. code-block:: c
    :caption: 代码清单:ETH-5 HAL_ETH_MspInit函数（文件LAN8720a.c）
    :name: 代码清单:ETH-5
    :linenos:

    /**
    * @brief  初始化ETH外设时钟，引脚.
    * @param  heth: ETH handle
    * @retval None
    */
    void HAL_ETH_MspInit(ETH_HandleTypeDef *heth)
    {
        ETH_GPIO_Config();
        /* 使能以太网时钟  */
        __HAL_RCC_ETH1MAC_CLK_ENABLE();
        __HAL_RCC_ETH1TX_CLK_ENABLE();
        __HAL_RCC_ETH1RX_CLK_ENABLE();
    }



.. code-block:: c
    :caption: 代码清单:ETH-6 LAN8720_Init函数（文件LAN8720a.c）
    :name: 代码清单:ETH-6
    :linenos:

    /**
    * @brief  初始化LAN8720A.
    * @param  heth: ETH handle
    * @retval HAL_StatusTypeDef：状态值
    */
    HAL_StatusTypeDef LAN8720_Init(ETH_HandleTypeDef *heth)
    {
        uint32_t phyreg = 0;
        uint32_t TIME_Out = 0;
        //软件复位LAN8720A
        if (HAL_ETH_WritePHYRegister(heth, LAN8720A_PHY_ADDRESS, PHY_BCR, PHY_RESET) != HAL_OK) {
            return HAL_ERROR;
        }
        //等待LAN8720A复位完成
        HAL_Delay(PHY_RESET_DELAY);

        if ((HAL_ETH_WritePHYRegister(heth, LAN8720A_PHY_ADDRESS, PHY_BCR, PHY_AUTONEGOTIATION)) !=
            HAL_OK) {
            return HAL_ERROR;
        }
        //等待LAN8720A写入完成
        HAL_Delay(0xFFF);
        do {
            HAL_ETH_ReadPHYRegister(heth, LAN8720A_PHY_ADDRESS, PHY_BSR, &phyreg);
            TIME_Out++;
            if (TIME_Out > PHY_READ_TO)
                return HAL_TIMEOUT;
        } while (((phyreg & PHY_AUTONEGO_COMPLETE) != PHY_AUTONEGO_COMPLETE));

        return HAL_OK;
    }


LAN8720_Init函数主要用于配置LAN8720A的寄存器。首先，向BCR寄存器的写入PHY_RESET值，调用HAL_Delay函数等待软件复位LAN8720A完成。
接着想BCR寄存器写入PHY_AUTONEGOTIATION，设置为PHY芯片的自适应功能，调用HAL_Delay函数等待LAN8720A写入完成。最后，
读取LAN8720A的BSR寄存器来确定器件器件是否正常工作，并将初始化器件的状态值返回。

ETH外设有专门实现数据搬运的DMA，可以在CPU完全不干预的情况下，用DMA描述符有效地将数据从源地址传送到目标地址。
STM32H743通过以下两个数据结构来实现通信：一个是控制和状态寄存器（CSR），另一个是DMA描述符列表和数据缓冲区。
所谓的描述符列表就是单片机的某一块内存单元，每一个描述符列表最多可以指向两个缓冲区。
调用MPU_Config函数来保护这部分的内存单元，以及区域内的DCache的读写方式。


.. code-block:: c
    :caption: 代码清单:ETH-7 DMA描述符定义（文件ethernetif.c）以及MPU外设配置（文件main.c）
    :name: 代码清单:ETH-7
    :linenos:

    //以太网Rx DMA描述符
    __attribute__((at(0x30040000))) ETH_DMADescTypeDef  DMARxDscrTab[ETH_RX_DESC_CNT];
    //以太网Tx DMA描述符
    __attribute__((at(0x30040060))) ETH_DMADescTypeDef  DMATxDscrTab[ETH_TX_DESC_CNT];
    //以太网接收缓冲区
    __attribute__((at(0x30040200))) uint8_t Rx_Buff[ETH_RX_DESC_CNT][ETH_MAX_PACKET_SIZE];


    /**
    * @brief  配置MPU外设
    * @param  None
    * @retval None
    */
    static void MPU_Config(void)
    {
        MPU_Region_InitTypeDef MPU_InitStruct;

        /* Disable the MPU */
        HAL_MPU_Disable();

        /* Configure the MPU attributes as Device not cacheable
            for ETH DMA descriptors */
        MPU_InitStruct.Enable = MPU_REGION_ENABLE;
        MPU_InitStruct.BaseAddress = 0x30040000;
        MPU_InitStruct.Size = MPU_REGION_SIZE_256B;
        MPU_InitStruct.AccessPermission = MPU_REGION_FULL_ACCESS;
        MPU_InitStruct.IsBufferable = MPU_ACCESS_BUFFERABLE;
        MPU_InitStruct.IsCacheable = MPU_ACCESS_NOT_CACHEABLE;
        MPU_InitStruct.IsShareable = MPU_ACCESS_NOT_SHAREABLE;
        MPU_InitStruct.Number = MPU_REGION_NUMBER0;
        MPU_InitStruct.TypeExtField = MPU_TEX_LEVEL0;
        MPU_InitStruct.SubRegionDisable = 0x00;
        MPU_InitStruct.DisableExec = MPU_INSTRUCTION_ACCESS_ENABLE;

        HAL_MPU_ConfigRegion(&MPU_InitStruct);

        /* Configure the MPU attributes as Cacheable write through
            for LwIP RAM heap which contains the Tx buffers */
        MPU_InitStruct.Enable = MPU_REGION_ENABLE;
        MPU_InitStruct.BaseAddress = 0x30044000;
        MPU_InitStruct.Size = MPU_REGION_SIZE_16KB;
        MPU_InitStruct.AccessPermission = MPU_REGION_FULL_ACCESS;
        MPU_InitStruct.IsBufferable = MPU_ACCESS_NOT_BUFFERABLE;
        MPU_InitStruct.IsCacheable = MPU_ACCESS_CACHEABLE;
        MPU_InitStruct.IsShareable = MPU_ACCESS_NOT_SHAREABLE;
        MPU_InitStruct.Number = MPU_REGION_NUMBER1;
        MPU_InitStruct.TypeExtField = MPU_TEX_LEVEL0;
        MPU_InitStruct.SubRegionDisable = 0x00;
        MPU_InitStruct.DisableExec = MPU_INSTRUCTION_ACCESS_ENABLE;

        HAL_MPU_ConfigRegion(&MPU_InitStruct);

        /* Enable the MPU */
        HAL_MPU_Enable(MPU_PRIVILEGED_DEFAULT);
    }


Low_level_init主要是初始化硬件外设，最终被 ethernetif_init函数调用。


.. code-block:: c
    :caption: 代码清单:ETH-8 low_level_init函数（文件ethernetif.c）
    :name: 代码清单:ETH-8
    :linenos:

    /**
    * @brief 在这个函数中初始化硬件.
    *        最终被ethernetif_init函数调用.
    *
    * @param netif已经初始化了这个以太网的lwip网络接口结构
    */
    static void low_level_init(struct netif *netif)
    {
        uint32_t idx = 0;
        //mac地址
    uint8_t macaddress[6]={MAC_ADDR0, MAC_ADDR1, MAC_ADDR2, MAC_ADDR3, MAC_ADDR4, MAC_ADDR5};

        EthHandle.Instance = ETH;
        EthHandle.Init.MACAddr = macaddress;
        //RMII模式
        EthHandle.Init.MediaInterface = HAL_ETH_RMII_MODE;
        //接受描述符
        EthHandle.Init.RxDesc = DMARxDscrTab;
        //发送描述符
        EthHandle.Init.TxDesc = DMATxDscrTab;
        //数据长度
        EthHandle.Init.RxBuffLen = ETH_RX_BUFFER_SIZE;

        /* 配置以太网外设 (GPIOs, clocks, MAC, DMA)*/
        HAL_ETH_Init(&EthHandle);

        /* 设置netif MAC 硬件地址长度 */
        netif->hwaddr_len = ETHARP_HWADDR_LEN;

        /* 设置netif MAC 硬件地址 */
        netif->hwaddr[0] =  MAC_ADDR0;
        netif->hwaddr[1] =  MAC_ADDR1;
        netif->hwaddr[2] =  MAC_ADDR2;
        netif->hwaddr[3] =  MAC_ADDR3;
        netif->hwaddr[4] =  MAC_ADDR4;
        netif->hwaddr[5] =  MAC_ADDR5;

        /* 设置netif最大传输单位 */
        netif->mtu = ETH_MAX_PAYLOAD;

        /* 接收广播地址和ARP流量  */
        netif->flags |= NETIF_FLAG_BROADCAST | NETIF_FLAG_ETHARP;

        for (idx = 0; idx < ETH_RX_DESC_CNT; idx ++) {
            HAL_ETH_DescAssignMemory(&EthHandle, idx, Rx_Buff[idx], NULL);

            /* Set Custom pbuf free function */
            rx_pbuf[idx].custom_free_function = pbuf_free_custom;
        }

        /* 设置发送配置结构体 */
        memset(&TxConfig, 0 , sizeof(ETH_TxPacketConfig));
        TxConfig.Attributes = ETH_TX_PACKETS_FEATURES_CSUM | ETH_TX_PACKETS_FEATURES_CRCPAD;
        //发送校验
        TxConfig.ChecksumCtrl = ETH_CHECKSUM_IPHDR_PAYLOAD_INSERT_PHDR_CALC;
        //CRC校验位
        TxConfig.CRCPadCtrl = ETH_CRC_PAD_INSERT;
        //初始化LAN8720A
        if (LAN8720_Init(&EthHandle) == HAL_OK) {
            ethernet_link_check_state(netif);
        }
    }


首先是ETH_HandleTypeDef结构体填充，关于结构体各个成员意义已在“ETH初始化结构体详解”作了分析。
然后调用系统函数HAL_ETH_Init初始化以太网外设。初始化相关描述符的列表，设置MAC地址和PHY芯片 LAN8720A。

Netif_Config函数一般在main函数中在LwIP_Init函数初始化完成后调用。注意，宏定义位于main.h文件中。


.. code-block:: c
    :caption: 代码清单:ETH-9 Netif_Config函数（文件app_ethernet.c）
    :name: 代码清单:ETH-9
    :linenos:

    #define DEST_IP_ADDR0   ((uint8_t)192U)
    #define DEST_IP_ADDR1   ((uint8_t)168U)
    #define DEST_IP_ADDR2   ((uint8_t)1U)
    #define DEST_IP_ADDR3   ((uint8_t)198U)

    #define DEST_PORT       ((uint16_t)7U)

    //静态IP地址: IP_ADDR0.IP_ADDR1.IP_ADDR2.IP_ADDR3
    #define IP_ADDR0   ((uint8_t) 192U)
    #define IP_ADDR1   ((uint8_t) 168U)
    #define IP_ADDR2   ((uint8_t) 0U)
    #define IP_ADDR3   ((uint8_t) 132U)

    //子网掩码
    #define NETMASK_ADDR0   ((uint8_t) 255U)
    #define NETMASK_ADDR1   ((uint8_t) 255U)
    #define NETMASK_ADDR2   ((uint8_t) 255U)
    #define NETMASK_ADDR3   ((uint8_t) 0U)

    //网关
    #define GW_ADDR0   ((uint8_t) 192U)
    #define GW_ADDR1   ((uint8_t) 168U)
    #define GW_ADDR2   ((uint8_t) 0U)
    #define GW_ADDR3   ((uint8_t) 1U)

    /**
    * @brief  建立网络接口
    * @param  None
    * @retval None
    */
    void Netif_Config(void)
    {
        ip_addr_t ipaddr;
        ip_addr_t netmask;
        ip_addr_t gw;

        IP_ADDR4(&ipaddr,IP_ADDR0,IP_ADDR1,IP_ADDR2,IP_ADDR3);
        IP_ADDR4(&netmask,NETMASK_ADDR0,NETMASK_ADDR1,NETMASK_ADDR2,NETMASK_ADDR3);
        IP_ADDR4(&gw,GW_ADDR0,GW_ADDR1,GW_ADDR2,GW_ADDR3);

        /* 添加网络接口 */
        netif_add(&gnetif, &ipaddr, &netmask, &gw, NULL, &ethernetif_init, &ethernet_input);

        /* 注册默认网络接口 */
        netif_set_default(&gnetif);

        if (netif_is_link_up(&gnetif)) {
            /* 当netif完全配置时，必须调用此函数 */
            netif_set_up(&gnetif);
        } else {
            /* 当netif链接断开时，必须调用此函数 */
            netif_set_down(&gnetif);
        }

    }

通过宏定义了远端IP和端口、MAC地址、静态IP地址、子网掩码、网关相关宏，可以根据实际情况修改。netif_add函数添加网络接口；netif_set_default注册默认网络接口。


.. code-block:: c
    :caption: 代码清单:ETH-10 User_notification函数（文件app_ethernet.c）
    :name: 代码清单:ETH-10
    :linenos:

    /**
    * @brief  通知用户有关网络接口配置状态
    * @param  netif: 网络接口
    * @retval None
    */
    void User_notification(struct netif *netif)
    {
        if (netif_is_up(netif)) {
            printf("Static IP: %d.%d.%d.%d\n",IP_ADDR0,IP_ADDR1,IP_ADDR2,IP_ADDR3);
            printf("NETMASK  : %d.%d.%d.%d\n",NETMASK_ADDR0,NETMASK_ADDR1,NETMASK_ADDR2,NETMASK_ADDR3);
            printf("Gateway  : %d.%d.%d.%d\n",GW_ADDR0,GW_ADDR1,GW_ADDR2,GW_ADDR3);
            LED_GREEN;
        } else {
            printf ("The network cable is not connected \n");
            LED_RED;
        }

    }



User_notification函数在网络接口配置完成后调用，通知用户有关网络接口配置状态。打印接口连接状态，LED指示连接状态。


.. code-block:: c
    :caption: 代码清单:ETH-11 lwip_Init函数（文件init.c）
    :name: 代码清单:ETH-11
    :linenos:

    /**
    * @ingroup lwip_nosys
    * 初始化所有模块.
    * 在NO_SYS模式下使用,否则使用tcpip_init（）。
    */
    void
    lwip_init(void)
    {
    #ifndef LWIP_SKIP_CONST_CHECK
        int a = 0;
        LWIP_UNUSED_ARG(a);
        LWIP_ASSERT("LWIP_CONST_CAST not implemented correctly. Check your lwIP port.", LWIP_CONST_CAST(void*, &a) == &a);
    #endif
    #ifndef LWIP_SKIP_PACKING_CHECK
        LWIP_ASSERT("Struct packing not implemented correctly. Check your lwIP port.", sizeof(struct packed_struct_test) ==
                    PACKED_STRUCT_TEST_EXPECTED_SIZE);
    #endif

        /* 模块初始化 */
        stats_init();
    #if !NO_SYS
        sys_init();
    #endif /* !NO_SYS */
        mem_init();
        memp_init();
        pbuf_init();
        netif_init();
    #if LWIP_IPV4
        ip_init();
    #if LWIP_ARP
        etharp_init();
    #endif /* LWIP_ARP */
    #endif /* LWIP_IPV4 */
    #if LWIP_RAW
        raw_init();
    #endif /* LWIP_RAW */
    #if LWIP_UDP
        udp_init();
    #endif /* LWIP_UDP */
    #if LWIP_TCP
        tcp_init();
    #endif /* LWIP_TCP */
    #if LWIP_IGMP
        igmp_init();
    #endif /* LWIP_IGMP */
    #if LWIP_DNS
        dns_init();
    #endif /* LWIP_DNS */
    #if PPP_SUPPORT
        ppp_init();
    #endif

    #if LWIP_TIMERS
        sys_timeouts_init();
    #endif /* LWIP_TIMERS */
    }


lwip_Init函数用于初始化LwIP协议栈，一般在main函数中调用。首先是内存相关初始化，mem_init函数是动态内存堆初始化，
memp_init函数是存储池初始化，LwIP是实现内存的高效利用，内部需要不同形式的内存管理模式。

pbuf 函数为预留的函数，目前是一个空操作。netif_init函数多播的时候用到，本例没有用到。后面的功能都是通过lwipopts.h进行裁剪。


.. code-block:: c
    :caption: 代码清单:ETH-12 ethernetif_input函数（文件ethernetif.c）
    :name: 代码清单:ETH-12
    :linenos:

    /**
    * @brief 当数据包准备好从接口读取时，应该调用此函数。
    *它使用应该处理来自网络接口的字节的实际接收的函数low_level_input。
    *然后确定接收到的分组的类型，并调用适当的输入功能。
    *
    * @param netif 以太网的lwip网络接口结构
    */
    void ethernetif_input(struct netif *netif)
    {
        err_t err;
        struct pbuf *p;

        /* 将接收到的数据包移动到新的pbuf中 */
        p = low_level_input(netif);

        /* 没有数据包可以读取，直接返回 */
        if (p == NULL) return;

        /* 到LwIP堆栈入口 */
        err = netif->input(p, netif);

        if (err != ERR_OK) {
            LWIP_DEBUGF(NETIF_DEBUG, ("ethernetif_input: IP input error\n"));
            pbuf_free(p);
            p = NULL;
        }
    }


ethernetif_input函数用于从以太网存储器读取一个以太网帧并将其发送给LwIP，它在接收到以太网帧时被调用，
它是直接调用low_level_input函数实现的，该函数定义在ethernetif.c文件中。


.. code-block:: c
    :caption: 代码清单:ETH-13 sys_check_timeouts函数（文件timeouts.c）
    :name: 代码清单:ETH-13
    :linenos:

    /**
    * @ingroup lwip_nosys
    * 处理NO_SYS==1超时 (即不使用tcpip_thread/sys_timeouts_mbox_fetch()）
    * 使用sys_now()函数，当超时到期时调用超时处理函数。
    * 必须定期从主循环中调用。
    */
    #if !NO_SYS && !defined __DOXYGEN__
    static
    #endif /* !NO_SYS */
    void
    sys_check_timeouts(void)
    {
        if (next_timeout) {
            struct sys_timeo *tmptimeout;
            u32_t diff;
            sys_timeout_handler handler;
            void *arg;
            u8_t had_one;
            u32_t now;

            now = sys_now();
            /* this cares for wraparounds */
            diff = now - timeouts_last_time;
            do {
                PBUF_CHECK_FREE_OOSEQ();
                had_one = 0;
                tmptimeout = next_timeout;
                if (tmptimeout && (tmptimeout->time <= diff)) {
                    /* timeout has expired */
                    had_one = 1;
                    timeouts_last_time += tmptimeout->time;
                    diff -= tmptimeout->time;
                    next_timeout = tmptimeout->next;
                    handler = tmptimeout->h;
                    arg = tmptimeout->arg;
    #if LWIP_DEBUG_TIMERNAMES
                    if (handler != NULL) {
                LWIP_DEBUGF(TIMERS_DEBUG, ("sct calling h=%s arg=%p\n",
                                        tmptimeout->handler_name, arg));
                    }
    #endif /* LWIP_DEBUG_TIMERNAMES */
                    memp_free(MEMP_SYS_TIMEOUT, tmptimeout);
                    if (handler != NULL) {
    #if !NO_SYS
            /* For LWIP_TCPIP_CORE_LOCKING, lock the core before calling the
                        timeout handler function. */
                        LOCK_TCPIP_CORE();
    #endif /* !NO_SYS */
                        handler(arg);
    #if !NO_SYS
                        UNLOCK_TCPIP_CORE();
    #endif /* !NO_SYS */
                    }
                    LWIP_TCPIP_THREAD_ALIVE();
                }
                /* repeat until all expired timers have been called */
            } while (had_one);
        }
    }


sys_check_timeouts函数是一个必须被无限循环调用的LwIP支持函数，一般在main函数的无限循环中调用，使用sys_now()函数，当超时到期时调用超时处理函数。

lwipopts.h文件存放一些宏定义，用于剪切LwIP功能，比如有无操作系统、内存空间分配、存储池分配、TCP功能、
DHCP功能、UDP功能选择等等。这里使用与ST官方例程相同配置即可。


.. code-block:: c
    :caption: 代码清单:ETH-14 main函数（mian.c文件）
    :name: 代码清单:ETH-14
    :linenos:

    /**
    * @brief  主函数
    * @param  无
    * @retval 无
    */
    int main(void)
    {
        /* 配置相应的内存单元为ETH的DMA描述符列表 */
        MPU_Config();

        /* Enable I-Cache */
        SCB_EnableICache();

        /* Enable D-Cache */
        SCB_EnableDCache();
        //将Cache设置write-through方式
        SCB->CACR|=1<<2;

        /* 配置系统时钟为480 MHz */
        SystemClock_Config();

        /* 初始化RGB彩灯 */
        LED_GPIO_Config();

        /* 初始化USART1 配置模式为 115200 8-N-1 */
        DEBUG_USART_Config();

        /* 初始化LwIP协议栈*/
        lwip_init();

        printf("LAN8720A Ethernet Demo\n");
        printf("LwIP版本：%s\n",LWIP_VERSION_STRING);

        printf("ping实验例程\n");

        printf("使用同一个局域网中的电脑ping开发板的地址，可进行测试\n");

        //IP地址和端口可在main.h文件修改
        printf("本地IP和端口: %d.%d.%d.%d\n",IP_ADDR0,IP_ADDR1,IP_ADDR2,IP_ADDR3);
        /* 网络接口配置 */
        Netif_Config();

        User_notification(&gnetif);
        while (1) {
            /* 从以太网缓冲区中读取数据包，交给LwIP处理 */
            ethernetif_input(&gnetif);
            /* 处理 LwIP 超时 */
            sys_check_timeouts();
        }
    }


首先是使能指令缓存、数据缓存，初始化系统时钟、LED指示灯、按键、调试串口，lwip_init 函数初始化LwIP协议栈。
通过Netif_Config函数配置网络接口；通过User_notification函数报告用户网络连接状态。进入无限循环函数，
调用ethernetif_input函数从以太网缓存中读取数据包并交给LwIP处理；调用sys_check_timeouts 函数处理LwIP超时。这两个函数必须在大循环中调用。

下载验证


保证开发板相关硬件连接正确，用USB线连接开发板“USB TO UART”接口跟电脑，在电脑端打开串口调试助手并配置好相关参数；
使用网线连接开发板网口跟路由器，这里要求电脑连接在同一个路由器上，之所以使用路由器是这样连接方便，电脑端无需更多操作步骤，
并且路由器可以提供DHCP服务器功能，而电脑不行的，最后在电脑端打开网络调试助手软件，并设置相关参数，
路由的网关跟main.h文件中相关宏定义是对应的，不同电脑设置情况可能不同。把编译好的程序下载到开发板。

在系统硬件初始化时串口调试助手会打印相关提示信息，等待初始化完成后可打开电脑端CMD窗口，输入ping命令测试开发板链路，
图 ping窗口_ 为链路正常情况，如果出现ping不同情况，检查网线连接。

.. image:: media/ETH020.png
    :align: center
    :name: ping窗口
    :alt: ping窗口


基于freertos移植LwIP实验
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

LwIP不仅能在裸机上运行，也能在操作系统环境下运行，而且在操作系统环境下，用户能使用NETCONN API 与Socket API编程，相比RAW API编程会更加简便。操作系统环境下，这意味着多线程环境，一般来说LwIP作为一个独立的处理线程运行，用户程序也独立为一个/多个线程，这样子在操作系统中就相互独立开，并且借助操作系统的IPC通信机制，更好地实现功能的需求。
LwIP在设计之初，设计者无法预测LwIP运行的环境是怎么样的，而且世界上操作系统那么多，根本没法统一，而如果LwIP要运行在操作系统环境中，那么就必须产生依赖，即LwIP需要依赖操作系统自身的通信机制，如信号量、互斥量、消息队列（邮箱）等，所以LwIP设计者在设计的时候就提供一套与操作系统相关的接口，由用户根据操作系统的不同进行移植，这样子就能降低耦合度，让LwIP内核不受其运行的环境影响，因为往往用户并不能完全了解内核的运作，所以只需要用户在移植的时候对LwIP提供的接口根据不同操作系统进行完善即可。

LwIP中添加操作系统
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

第一步：拷贝FreeRTOS源码到工程文件夹


我们先把无操作性移植的代码拷贝过来，再往工程中添加操作系统的源码，操作系统的源码可以从我们对外发布的例程中获取，也可以从官网获取，此处以FreeRTOS为例进行移植操作。
首先将FreeRTOS源码拷贝到工程文件夹下，具体见

.. image:: media/1.png
    :align: center
    :name: none
    :alt: none


第二步：添加FreeRTOS源码到工程组文件夹


在上一步我们只是将FreeRTOS的源码放到了本地工程目录下，还没有添加到开发环境里面的组文件夹里面，FreeRTOS也就没有移植到我们的工程中去。
接下来我们在开发环境里面新建FreeRTOS/src和FreeRTOS/port两个组文件夹，其中FreeRTOS/src用于存放src文件夹的所有内容，FreeRTOS/port用于存放port\MemMang文件夹与port\RVDS\ARM_CM？文件夹的内容，“？”表示3、4或者7，具体选择哪个得看你使用的是野火哪个型号的STM32开发板，具体见 表1_。

.. image:: media/2.png
    :align: center
    :name: 表1
    :alt: 表1


然后我们将工程文件中FreeRTOS的内容添加到工程中去，按照已经新建的分组添加我们的FreeRTOS工程源码。
在FreeRTOS/port分组中添加MemMang文件夹中的文件只需选择其中一个即可，我们选择“heap_4.c”，这是FreeRTOS的一个内存管理源码文件。同时，需要根据自己的开发板型号在FreeRTOS\port\RVDS\ARM_CM?中选择，“？”表示3、4或者7，具体选择哪个得看你使用的是野火哪个型号的STM32开发板，具体见表8 1。
至此我们的FreeRTOS添加到工程中就已经完成，完成的效果具体见 图2_。


.. image:: media/3.png
    :align: center
    :name: 图2
    :alt: 图2



第三步：指定FreeRTOS头文件的路径


FreeRTOS的源码已经添加到开发环境的组文件夹下面，编译的时候需要为这些源文件指定头文件的路径，不然编译会报错。FreeRTOS的源码里面只有FreeRTOS\include和FreeRTOS\port\RVDS\ARM_CM？这两个文件夹下面有头文件，只需要将这两个头文件的路径在开发环境里面指定即可。同时我们还将FreeRTOSConfig.h这个头文件拷贝到了工程根目录下的user文件夹下，所以user的路径也要加到开发环境里面。FreeRTOS头文件的路径添加完成后的效果具体见 图3_。


.. image:: media/4.png
    :align: center
    :name: 图3
    :alt: 图3


至此，FreeRTOS的整体工程基本移植完毕，我们也不需要修改FreeRTOS配置文件，直接使用我们FreeRTOS例程中的头文件即可。

第四步： 修改stm32h7xx_it.c

SysTick中断服务函数是一个非常重要的函数，FreeRTOS所有跟时间相关的事情都在里面处理，SysTick就是FreeRTOS的一个心跳时钟，驱动着FreeRTOS的运行，就像人的心跳一样，
假如没有心跳，我们就相当于“死了”，同样的，FreeRTOS没有了心跳，那么它就会卡死在某个地方，不能进行任务调度，不能运行任何的东西，因此我们需要实现一个FreeRTOS的心跳时钟，
FreeRTOS帮我们实现了SysTick的启动的配置：在port.c文件中已经实现vPortSetupTimerInterrupt()函数，并且FreeRTOS通用的SysTick中断服务函数也实现了：
在port.c文件中已经实现xPortSysTickHandler()函数，所以移植的时候只需要我们在stm32f4xx_it.c文件中实现我们对应（STM32）平台上的SysTick_Handler()函数即可。
FreeRTOS为开发者考虑得特别多，PendSV_Handler()与SVC_Handler()这两个很重要的函数都帮我们实现了，在在port.c文件中已经实现xPortPendSVHandler()与vPortSVCHandler()函数，
防止我们自己实现不了，那么在stm32f4xx_it.c中就需要我们注释掉或者删除掉PendSV_Handler()与SVC_Handler()这两个函数了，具体实现见  代码清单:ETH-16_。

.. code-block:: c
    :caption: 代码清单:ETH-16 宏定义
    :name: 代码清单:ETH-16
    :linenos:

    /* Includes ----------------------------------------------------------*/
    #include "main.h"
    #include "stm32h7xx_it.h"
    #include "./usart/bsp_debug_usart.h"
    #include "FreeRTOS.h" //FreeRTOS 使用
    #include "task.h"
    
    /* Private typedef ----------------------------------------------*/
    /* Private define ------------------------------------------------*/
    /* Private macro -------------------------------------------------*/
    /* Private variables --------------------------------------------*/
    /* Private function prototypes -------------------------------------*/
    /* Private functions ---------------------------------------------*/
    /*******************************************************************/
    /* Cortex-M4 Processor Exceptions Handlers */
    /*****************************************************************/
    /**
    * @brief This function handles NMI exception.
    * @param None
    * @retval None
    */
    void NMI_Handler(void)
    {
    }
    /**
    * @brief This function handles Hard Fault exception.
    * @param None
    * @retval None
    */
    //void HardFault_Handler(void)
    //{
    // /* Go to infinite loop when Hard Fault exception occurs */
    // while (1)
    // {
    // }
    //}
    /**
    * @brief This function handles Memory Manage exception.
    * @param None
    * @retval None
    */
    void MemManage_Handler(void)
    {
    /* Go to infinite loop when Memory Manage exception occurs */
    while (1)
    {
    }
    }
    /**
    * @brief This function handles Bus Fault exception.
    * @param None
    * @retval None
    */
    void BusFault_Handler(void)
    {
    /* Go to infinite loop when Bus Fault exception occurs */
    while (1)
    {
    }
    }
    /**
    * @brief This function handles Usage Fault exception.
    * @param None
    * @retval None
    */
    void UsageFault_Handler(void)
    {
    /* Go to infinite loop when Usage Fault exception occurs */
    while (1)
    {
    }
    }
    /**
    * @brief This function handles Debug Monitor exception.
    * @param None
    * @retval None
    */
    void DebugMon_Handler(void)
    {
    }
    /**
    * @brief This function handles SysTick Handler.
    * @param None
    * @retval None
    */
    extern void xPortSysTickHandler(void);
    void SysTick_Handler(void)
    {
    uint32_t ulReturn;
    /* 进入临界段，临界段可以嵌套 */
    ulReturn = taskENTER_CRITICAL_FROM_ISR();
    HAL_IncTick();
    #if (INCLUDE_xTaskGetSchedulerState == 1 )
    if (xTaskGetSchedulerState() != taskSCHEDULER_NOT_STARTED)
    {
    #endif /* INCLUDE_xTaskGetSchedulerState */
    xPortSysTickHandler();
    #if (INCLUDE_xTaskGetSchedulerState == 1 )
    }
    #endif /* INCLUDE_xTaskGetSchedulerState */
    /* 退出临界段 */
    taskEXIT_CRITICAL_FROM_ISR( ulReturn );
    }


至此，将FreeRTOS添加到LwIP裸机工程中的步骤就基本完成了，编译的时候就基本也不会有错，下面就正式使用这个工程进行LwIP与操作系统的移植，因为还需要根据操作系统的特性修改很多接口文件。

lwipopts.h文件需要加入的配置
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
在前面的章节也说了lwipopts.h文件的作用，而此刻在操作系统中移植，我们首先要将添加了操作系统的工程拿过来，把lwipopts.h文件修改一下，该文件最重要的宏定义就是NO_SYS，
我们把它定义为0就表示使用操作系统，当然，在使用操作系统的时候我们一般都会使用NETCONN API 与Socket API编程，那么就需要将宏LWIP_NETCONN与LWIP_SOCKET定义为1，
表示使能这两种API编程，lwipopts.h简单修改一下即可，然后再添加一下线程运行的一些宏定义，必须修改的部分具体见 代码清单:ETH-17_，而其他宏定义是根据实际情况进行修改即可

.. code-block:: c
    :caption: 需要加入的配置
    :name: 代码清单:ETH-17
    :linenos:

    #ifndef __LWIPOPTS_H__
    #define __LWIPOPTS_H__
    /**
    * SYS_LIGHTWEIGHT_PROT==1: if you want inter-task protection for certai
    * critical regions during buffer allocation, deallocation and memory
    * allocation and deallocation.
    */
    #define SYS_LIGHTWEIGHT_PROT 1
    /**
    * NO_SYS==1: Provides VERY minimal functionality. Otherwise,
    * use lwIP facilities.
    */
    #define NO_SYS 0
    /**
    * NO_SYS_NO_TIMERS==1: Drop support for sys_timeout when NO_SYS==1
    * Mainly for compatibility to old versions.
    */
    #define NO_SYS_NO_TIMERS 0
    /* ---------- Memory options ---------- */
    /* MEM_ALIGNMENT: should be set to the alignment of the CPU for which
    lwIP is compiled. 4 byte alignment -> define MEM_ALIGNMENT to 4, 2
    byte alignment -> define MEM_ALIGNMENT to 2. */
    #define MEM_ALIGNMENT 4
    /* MEM_SIZE: the size of the heap memory. If the application will send
    a lot of data that needs to be copied, this should be set high. */
    #define MEM_SIZE (15*1024)
    /* MEMP_NUM_PBUF: the number of memp struct pbufs. If the application
    sends a lot of data out of ROM (or other static memory), this
    should be set high. */
    #define MEMP_NUM_PBUF 25
    /* MEMP_NUM_UDP_PCB: the number of UDP protocol control blocks. One
    per active UDP "connection". */
    #define MEMP_NUM_UDP_PCB 4
    /* MEMP_NUM_TCP_PCB: the number of simulatenously active TCP
    connections. */
    #define MEMP_NUM_TCP_PCB 6
    /* MEMP_NUM_TCP_PCB_LISTEN: the number of listening TCP
    connections. */
    #define MEMP_NUM_TCP_PCB_LISTEN 6
    /* MEMP_NUM_TCP_SEG: the number of simultaneously queued TCP
    segments. */
    #define MEMP_NUM_TCP_SEG 150
    /* MEMP_NUM_SYS_TIMEOUT: the number of simulateously active
    timeouts. */
    #define MEMP_NUM_SYS_TIMEOUT 6
    /* ---------- Pbuf options ---------- */
    /* PBUF_POOL_SIZE: the number of buffers in the pbuf pool. */
    #define PBUF_POOL_SIZE 45
    /* PBUF_POOL_BUFSIZE: the size of each pbuf in the pbuf pool. */
    #define PBUF_POOL_BUFSIZE \
    LWIP_MEM_ALIGN_SIZE(TCP_MSS+40+PBUF_LINK_ENCAPSULATION_HLEN+PBUF_LINK_HLEN)
    /* ---------- TCP options ---------- */
    #define LWIP_TCP 1
    #define TCP_TTL 255
    /* Controls if TCP should queue segments that arrive out of
    order. Define to 0 if your device is low on memory. */
    #define TCP_QUEUE_OOSEQ 0
    /* TCP Maximum segment size. */
    #define TCP_MSS (1500 - 40)
    /* TCP sender buffer space (bytes). */
    #define TCP_SND_BUF (10*TCP_MSS)
    /* TCP_SND_QUEUELEN: TCP sender buffer space (pbufs). This must be at least
    as much as (2 * TCP_SND_BUF/TCP_MSS) for things to work. */
    #define TCP_SND_QUEUELEN (8* TCP_SND_BUF/TCP_MSS)
    /* TCP receive window. */
    #define TCP_WND (11*TCP_MSS)
    /* ---------- ICMP options ---------- */
    #define LWIP_ICMP 1
    /* ---------- DHCP options ---------- */
    /* Define LWIP_DHCP to 1 if you want DHCP configuration of
    interfaces. DHCP is not implemented in lwIP 0.5.1, however, so
    turning this on does currently not work. */
    #define LWIP_DHCP 1
    /* ---------- UDP options ---------- */
    #define LWIP_UDP 1
    #define UDP_TTL 255
    /* ---------- Statistics options ---------- */
    #define LWIP_STATS 0
    #define LWIP_PROVIDE_ERRNO 1
    /* ---------- link callback options ---------- */
    /* LWIP_NETIF_LINK_CALLBACK==1: Support a callback function from an
    erface
    * whenever the link changes (i.e., link down)
    */
    #define LWIP_NETIF_LINK_CALLBACK 0
    /*
    --------------------------------------
    ---------- Checksum options ----------
    --------------------------------------
    */
    /* The STM32F4x7 allows computing and verifying the IP,
    UDP, TCP and ICMP checksums by hardware:
    - To use this feature let the following define uncommented.
    - To disable it and process by CPU comment the the checksum.
    */
    #define CHECKSUM_BY_HARDWARE
    #ifdef CHECKSUM_BY_HARDWARE
    /* CHECKSUM_GEN_IP==0: Generate checksums by hardware for outgoing IP packets.*/
    #define CHECKSUM_GEN_IP 0
    /* CHECKSUM_GEN_UDP==0: Generate checksums by hardware for outgoing UDP packets.*/
    #define CHECKSUM_GEN_UDP 0
    /* CHECKSUM_GEN_TCP==0: Generate checksums by hardware for outgoing TCP packets.*/
    #define CHECKSUM_GEN_TCP 0
    /* CHECKSUM_CHECK_IP==0: Check checksums by hardware for incoming IP packets.*/
    #define CHECKSUM_CHECK_IP 0
    /* CHECKSUM_CHECK_UDP==0: Check checksums by hardware for incoming UDP packets.*/
    #define CHECKSUM_CHECK_UDP 0
    /* CHECKSUM_CHECK_TCP==0: Check checksums by hardware for incoming TCP packets.*/
    #define CHECKSUM_CHECK_TCP 0
    /*CHECKSUM_CHECK_ICMP==0: Check checksums by hardware for incoming ICMP packets.*/
    #define CHECKSUM_GEN_ICMP 0
    #else
    /* CHECKSUM_GEN_IP==1: Generate checksums in software for outgoing IP packets.*/
    #define CHECKSUM_GEN_IP 1
    /* CHECKSUM_GEN_UDP==1: Generate checksums in software for outgoing UDP packets.*/
    #define CHECKSUM_GEN_UDP 1
    /* CHECKSUM_GEN_TCP==1: Generate checksums in software for outgoing TCP packets.*/
    #define CHECKSUM_GEN_TCP 1
    /* CHECKSUM_CHECK_IP==1: Check checksums in software for incoming IP packets.*/
    #define CHECKSUM_CHECK_IP 1
    /* CHECKSUM_CHECK_UDP==1: Check checksums in software for incoming UDP packets.*/
    #define CHECKSUM_CHECK_UDP 1
    /* CHECKSUM_CHECK_TCP==1: Check checksums in software for incoming TCP packets.*/
    #define CHECKSUM_CHECK_TCP 1
    /*CHECKSUM_CHECK_ICMP==1: Check checksums by hardware for incoming ICMP packets.*/
    #define CHECKSUM_GEN_ICMP 1
    #endif
    /*
    ----------------------------------------------
    ---------- Sequential layer options ----------
    ----------------------------------------------
    */
    /**
    * LWIP_NETCONN==1: Enable Netconn API (require to use api_lib.c)
    */
    #define LWIP_NETCONN 1
    /*
    ------------------------------------
    ---------- Socket options ----------
    ------------------------------------
    */
    /**
    * LWIP_SOCKET==1: Enable Socket API (require to use sockets.c)
    */
    #define LWIP_SOCKET 1
    /*
    ---------------------------------
    ---------- OS options ----------
    ---------------------------------
    */
    #define DEFAULT_UDP_RECVMBOX_SIZE 10
    #define DEFAULT_TCP_RECVMBOX_SIZE 10
    #define DEFAULT_ACCEPTMBOX_SIZE 10
    #define DEFAULT_THREAD_STACKSIZE 1024
    #define TCPIP_THREAD_NAME "lwip"
    #define TCPIP_THREAD_STACKSIZE 512
    #define TCPIP_MBOX_SIZE 8
    #define TCPIP_THREAD_PRIO 3

操作系统环境下， LwIP移植的核心就是编写与操作系统相关的接口文件sys_arch.c和sys_arch.h，这两个文件可以自己创建也可以从contrib包中获取，
路径分别为“contrib-2.1.0\ports\freertos”与“contrib-2.1.0\ports\freertos\include\arch”，用户在移植的时候必须根据操作系统的功能为协议栈提供相应的接口，
如邮箱（因为本次移植以FreeRTOS为例子，FreeRTOS中没有邮箱这种概念，但是可以使用消息队列替代，为了迎合LwIP中的命名，下文统一采用邮箱表示）、信号量、互斥量等，
这些IPC通信机制是保证内核与上层API接口通信的基本保障，也是内核实现管理的继承，同时在sys.h文件中声明了用户需要实现的所有函数框架，这些函数具体见 表2_。

.. image:: media/6.png
    :align: center
    :name: 表2
    :alt: 表2

.. image:: media/7.png
    :align: center
    :name: 表2_1
    :alt: 表2_1

看到那么多函数，是不是头都大了，其实这些函数的实现都是很简单的，首先讲解一下邮箱函数的实现。在LwIP中，用户代码与协议栈内部之间是通过邮箱进行数据的交互的，邮箱本质上就是一个指向数据的指针，API将指针传递给内核，内核通过这个指针访问数据，然后去处理，反之内核将数据传递给用户代码也是通过邮箱将一个指针进行传递。
在操作系统环境下，LwIP会作为一个线程运行，线程的名字叫tcpip_thread，在初始化LwIP的时候，内核就会自动创建这个线程，并且在线程运行的时候阻塞在邮箱上，等待数据进行处理，这个邮箱数据的来源可能在底层网卡接收到的数据或者上层应用程序的数据，总之，tcpip_thread线程在获取到邮箱中的数据时候，就会退出阻塞态，去处理数据，在处理完毕数据后又进入阻塞态中等待数据的到来，如此反复。
信号量与互斥量的实现为内核提供同步与互斥的机制，比如当用户想要发送一个数据的时候，就会调用上层API接口，API接口就会去先发送一个数据给内核去处理，然后尝试获取一个信号量，因为此时是没有信号量的，所以就会阻塞用户线程；内核在知道用户想要发送数据后，就会调用对应的网卡去发送数据，当数据发送完成后就释放一个信号量告知用户线程发送完成，这样子用户线程就得以继续执行。
所以这些函数的接口都必须由用户实现，下面具体看看这些函数的实现，具体见 代码清单:ETH-18_



.. code-block:: c
    :caption: 代码清单:ETH-18
    :name: 代码清单:ETH-18
    :linenos:

    #include "debug.h"

    #include <lwip/opt.h>
    #include <lwip/arch.h>

    #include "tcpip.h"
    #include "lwip/init.h"
    #include "lwip/netif.h"
    #include "lwip/sio.h"
    #include "ethernetif.h"

    #if !NO_SYS
    #include "sys_arch.h"
    #endif
    #include <lwip/stats.h>
    #include <lwip/debug.h>
    #include <lwip/sys.h>

    #include <string.h>

    int errno;


    u32_t lwip_sys_now;

    struct sys_timeouts
    {
        struct sys_timeo *next;
    };

    struct timeoutlist
    {
        struct sys_timeouts timeouts;
        xTaskHandle pid;
    };

    #define SYS_THREAD_MAX 4

    static struct timeoutlist s_timeoutlist[SYS_THREAD_MAX];

    static u16_t s_nextthread = 0;

    u32_t
    sys_jiffies(void)
    {
        lwip_sys_now = xTaskGetTickCount();
        return lwip_sys_now;
    }

    u32_t
    sys_now(void)
    {
        lwip_sys_now = xTaskGetTickCount();
        return lwip_sys_now;
    }

    void
    sys_init(void)
    {
        int i;
        // Initialize the the per-thread sys_timeouts structures
        // make sure there are no valid pids in the list
        for (i = 0; i < SYS_THREAD_MAX; i++)
        {
            s_timeoutlist[i].pid = 0;
            s_timeoutlist[i].timeouts.next = NULL;
        }
        // keep track of how many threads have been created
        s_nextthread = 0;
    }

    struct sys_timeouts *sys_arch_timeouts(void)
    {
        int i;
        xTaskHandle pid;
        struct timeoutlist *tl;
        pid = xTaskGetCurrentTaskHandle( );
        for (i = 0; i < s_nextthread; i++)
        {
            tl = &(s_timeoutlist[i]);
            if (tl->pid == pid)
            {
                return &(tl->timeouts);
            }
        }
        return NULL;
    }

    sys_prot_t sys_arch_protect(void)
    {
        vPortEnterCritical();         //进入临界段
        return 1;
    }

    void sys_arch_unprotect(sys_prot_t pval)
    {
        ( void ) pval;
        vPortExitCritical();      //退出临界段
    }
    
    #if !NO_SYS
    
    err_t
    sys_sem_new(sys_sem_t *sem, u8_t count)
    {
        /* 创建 sem */
        if (count <= 1)
        {
            *sem = xSemaphoreCreateBinary();      //创建二值信号量
            if (count == 1)
            {
                sys_sem_signal(*sem);  //新创建的信号量是无效的，需要释放一个信号量
            }
        }
        else
            *sem = xSemaphoreCreateCounting(count,count); //创建计数信号量
    
    #if SYS_STATS
        ++lwip_stats.sys.sem.used;
        if (lwip_stats.sys.sem.max < lwip_stats.sys.sem.used)
        {
            lwip_stats.sys.sem.max = lwip_stats.sys.sem.used;
        }
    #endif /* SYS_STATS */
    
        if (*sem != SYS_SEM_NULL)
            return ERR_OK;          //创建成功返回ERR_OK
        else
        {
    #if SYS_STATS
            ++lwip_stats.sys.sem.err;
    #endif /* SYS_STATS */
            printf("[sys_arch]:new sem fail!\n");
            return ERR_MEM;
        }
    }
    
    void
    sys_sem_free(sys_sem_t *sem)
    {
    #if SYS_STATS
        --lwip_stats.sys.sem.used;
    #endif /* SYS_STATS */
        /* 删除 sem */
        vSemaphoreDelete(*sem);   //删除一个信号量
        *sem = SYS_SEM_NULL;  //删除之后置空
    }
    
    
    int sys_sem_valid(sys_sem_t *sem)
    {
        return (*sem != SYS_SEM_NULL);   //返回信号量是否有效
    }
    
    
    void
    sys_sem_set_invalid(sys_sem_t *sem)
    {
        *sem = SYS_SEM_NULL;    //信号量设置为无效
    }
    
    /*
    如果timeout参数不为零，则返回值为
    等待信号量所花费的毫秒数。如果
    信号量未在指定时间内发出信号，返回值为
    SYS_ARCH_TIMEOUT。如果线程不必等待信号量
    该函数返回零。 */
    u32_t
    sys_arch_sem_wait(sys_sem_t *sem, u32_t timeout)
    {
        u32_t wait_tick = 0;
        u32_t start_tick = 0 ;

        //看看信号量是否有效
        if (*sem == SYS_SEM_NULL)
            return SYS_ARCH_TIMEOUT;

        //首先获取开始等待信号量的时钟节拍
        start_tick = xTaskGetTickCount();

        //timeout != 0，需要将ms换成系统的时钟节拍
        if (timeout != 0)
        {
            //将ms转换成时钟节拍
            wait_tick = timeout / portTICK_PERIOD_MS;
            if (wait_tick == 0)
                wait_tick = 1;
        }
        else
            wait_tick = portMAX_DELAY;  //一直阻塞

        //等待成功，计算等待的时间，否则就表示等待超时
        if (xSemaphoreTake(*sem, wait_tick) == pdTRUE)
            return ((xTaskGetTickCount()-start_tick)*portTICK_RATE_MS);
        else
            return SYS_ARCH_TIMEOUT;
    }

    void
    sys_sem_signal(sys_sem_t *sem)
    {
        if (xSemaphoreGive( *sem ) != pdTRUE)   //释放信号量
            printf("[sys_arch]:sem signal fail!\n");
    }

    err_t
    sys_mutex_new(sys_mutex_t *mutex)
    {
        /* 创建 sem */
        *mutex = xSemaphoreCreateMutex(); //创建互斥量
        if (*mutex != SYS_MRTEX_NULL)
            return ERR_OK;      //创建成功返回ERR_OK
        else
        {
            printf("[sys_arch]:new mutex fail!\n");
            return ERR_MEM;
        }
    }

    void
    sys_mutex_free(sys_mutex_t *mutex)
    {
        vSemaphoreDelete(*mutex);   //删除互斥量
    }

    void
    sys_mutex_set_invalid(sys_mutex_t *mutex)
    {
        *mutex = SYS_MRTEX_NULL;    //设置互斥量为无效
    }

    void
    sys_mutex_lock(sys_mutex_t *mutex)
    {
        xSemaphoreTake(*mutex,/* 互斥量句柄 */
                    portMAX_DELAY); /* 等待时间 */
    }

    void
    sys_mutex_unlock(sys_mutex_t *mutex)
    {
        xSemaphoreGive( *mutex );//给出互斥量
    }

    sys_thread_t
    sys_thread_new(const char *name, lwip_thread_fn function,
            void *arg, int stacksize, int prio)
    {
        sys_thread_t handle = NULL;
        BaseType_t xReturn = pdPASS;
        /* 创建一个线程 */
        xReturn = xTaskCreate((TaskFunction_t )function,  /* 线程入口函数 */
                            (const char*    )name,/* 线程名字 */
                            (uint16_t       )stacksize,  /* 线程栈大小 */
                            (void*          )arg,/* 线程入口函数参数 */
                            (UBaseType_t    )prio, /* 线程的优先级 */
                            (TaskHandle_t*  )&handle);/* 线程控制块指针 */
        if (xReturn != pdPASS)
        {
            printf("[sys_arch]:create task fail!err:%#lx\n",xReturn);
            return NULL;
        }
        return handle;
    }

    err_t
    sys_mbox_new(sys_mbox_t *mbox, int size)
    {
        /* 创建一个邮箱 */
        *mbox = xQueueCreate((UBaseType_t ) size,/* 邮箱的长度 */
                            (UBaseType_t ) sizeof(void *));/* 消息的大小 */
    #if SYS_STATS
        ++lwip_stats.sys.mbox.used;
        if (lwip_stats.sys.mbox.max < lwip_stats.sys.mbox.used)
        {
            lwip_stats.sys.mbox.max = lwip_stats.sys.mbox.used;
        }
    #endif /* SYS_STATS */
        if (NULL == *mbox)
            return ERR_MEM;   // 创建成功返回ERR_OK

        return ERR_OK;
    }

    void
    sys_mbox_free(sys_mbox_t *mbox)
    {
        if ( uxQueueMessagesWaiting( *mbox ) )
        {
            /* Line for breakpoint.  Should never break here! */
            portNOP();
    #if SYS_STATS
            lwip_stats.sys.mbox.err++;
    #endif /* SYS_STATS */
        }

        vQueueDelete(*mbox);    //删除一个邮箱

    #if SYS_STATS
        --lwip_stats.sys.mbox.used;
    #endif /* SYS_STATS */
    }

    int sys_mbox_valid(sys_mbox_t *mbox)
    {
        if (*mbox == SYS_MBOX_NULL)     //判断邮箱是否有效
            return 0;
        else
            return 1;
    }

    void
    sys_mbox_set_invalid(sys_mbox_t *mbox)
    {
        *mbox = SYS_MBOX_NULL;  //设置有效为无效状态
    }

    void
    sys_mbox_post(sys_mbox_t *q, void *msg)
    {
        while (xQueueSend( *q, /* 邮箱的句柄 */
                        &msg,/* 发送的消息内容 */
                        portMAX_DELAY) != pdTRUE); /* 等待时间 */
    }

    err_t
    sys_mbox_trypost(sys_mbox_t *q, void *msg)
    {
        if (xQueueSend(*q,&msg,0) == pdPASS) //尝试发送一个消息，非阻塞发送
            return ERR_OK;
        else
            return ERR_MEM;
    }

    err_t
    sys_mbox_trypost_fromisr(sys_mbox_t *q, void *msg)
    {
        uint32_t ulReturn;
        err_t err = ERR_MEM;
        BaseType_t pxHigherPriorityTaskWoken;

        /* 进入临界段，临界段可以嵌套 */
        ulReturn = taskENTER_CRITICAL_FROM_ISR();

        if (xQueueSendFromISR(*q,&msg,&pxHigherPriorityTaskWoken)==pdPASS)
        {
            err = ERR_OK;
        }
        //如果需要的话进行一次线程切换
        portYIELD_FROM_ISR(pxHigherPriorityTaskWoken);

        /* 退出临界段 */
        taskEXIT_CRITICAL_FROM_ISR( ulReturn );

        return err;
    }

    u32_t
    sys_arch_mbox_fetch(sys_mbox_t *q, void **msg, u32_t timeout)
    {
        void *dummyptr;
        u32_t wait_tick = 0;
        u32_t start_tick = 0 ;

        if ( msg == NULL )  //看看存储消息的地方是否有效
            msg = &dummyptr;

        //首先获取开始等待信号量的时钟节拍
        start_tick = sys_now();

        //timeout != 0，需要将ms换成系统的时钟节拍
        if (timeout != 0)
        {
            //将ms转换成时钟节拍
            wait_tick = timeout / portTICK_PERIOD_MS;
            if (wait_tick == 0)
                wait_tick = 1;
        }
        //一直阻塞
        else
            wait_tick = portMAX_DELAY;

        //等待成功，计算等待的时间，否则就表示等待超时
        if (xQueueReceive(*q,&(*msg), wait_tick) == pdTRUE)
            return ((sys_now() - start_tick)*portTICK_PERIOD_MS);
        else
        {
            *msg = NULL;
            return SYS_ARCH_TIMEOUT;
        }
    }

    u32_t
    sys_arch_mbox_tryfetch(sys_mbox_t *q, void **msg)
    {
        void *dummyptr;
        if ( msg == NULL )
            msg = &dummyptr;

        //等待成功，计算等待的时间
        if (xQueueReceive(*q,&(*msg), 0) == pdTRUE)
            return ERR_OK;
        else
            return SYS_MBOX_EMPTY;
    }

    #if LWIP_NETCONN_SEM_PER_THREAD
    #error LWIP_NETCONN_SEM_PER_THREAD==1 not supported
    #endif /* LWIP_NETCONN_SEM_PER_THREAD */

    #endif /* !NO_SYS */



这些函数都是对操作系统的IPC通信机制进行简单的封装，在这里用户只需要稍微注意一下sys_arch_sem_wait()函数与sys_arch_mbox_fetch()函数，因为LwIP中使用的时间是以毫秒（ms）为单位的，
而操作系统中则以时钟节拍（tick）为单位，那么在返回等待信号量或者邮箱所使用的时间就是要转换成ms，而操作系统并未提供等待这些信息的时间，那么我们可以使用一个折中的方法，
在获取的时候开始记录时间戳，在获取结束后再次记录一次时间戳，两次时间戳相减就得到等待的时间，但是需要将这些时间（tick）转换为毫秒，这种做法当然是不精确的，但是对LwIP来说影响不大。

网卡底层的编写
^^^^^^^^^^^^^^^^^^

在无操作性移植的时候，我们的网卡收发数据就是单纯的收发数据，ethernetif_input()函数就是处理接收网卡数据的，但是使用了操作系统的话，我们一般将接收数据函数独立成为一个网卡接收线程，
这样子在收到数据的时候才去处理数据，然后递交给内核线程，所以我们只需要稍作修改即可，将函数转换成线程就行了，并且在初始化网卡的时候创建网卡接收线程。当然，我们也能将发送函数独立成一个线程，
我们暂时没有必要去处理它，此处只创建一个网卡接收线程，具体见 代码清单:ETH-19_。

.. code-block:: c
    :caption: 代码清单:ETH-19 调试信息输出定于
    :name: 代码清单:ETH-19
    :linenos:

    void ethernetif_input(void *pParams)
    {
        struct netif *netif;
        struct pbuf *p = NULL;
        netif = (struct netif*) pParams;
        LWIP_DEBUGF(NETIF_DEBUG, ("ethernetif_input: IP input error\n"));

        while (1)
        {
            if (xSemaphoreTake( s_xSemaphore, portMAX_DELAY ) == pdTRUE)
            {
                /* move received packet into a new pbuf */
                taskENTER_CRITICAL();
                p = low_level_input(netif);
                taskEXIT_CRITICAL();
                /* points to packet payload, which starts with an Ethernet header */
                if (p != NULL)
                {
                    taskENTER_CRITICAL();
                    /* full packet send to tcpip_thread to process */
                    if (netif->input(p, netif) != ERR_OK)
                    {
                    LWIP_DEBUGF(NETIF_DEBUG, ("ethernetif_input: IP input error\n"));
                        pbuf_free(p);
                        p = NULL;
                    }
                    taskEXIT_CRITICAL();
                }
            }
        }
    }



在网卡接收线程中需要留意一下以下内容：网卡接收线程是需要通过信号量机制去接收数据的，一般来说我们都是使用中断的方式去获取网络数据包，当产生中断的时候，
我们一般不会在中断中处理数据，而是告诉对应的线程去处理，也就是我们的网卡接收线程去处理数据，那么就会通过信号量进行同步，当网卡接收到了数据就会产生中断释放一个信号量，
然后线程从阻塞中恢复，去获取网卡的数据并且向上层递交。
当然我们还需要在中断中对网卡底层进行编写，具体见 代码清单:ETH-20_



.. code-block:: c
    :caption: 代码清单:ETH-20 sys_now函数
    :name: 代码清单:ETH-20
    :linenos:

    void ETH_IRQHandler(void)
    {
        uint32_t ulReturn;
        /* 进入临界段，临界段可以嵌套 */
        ulReturn = taskENTER_CRITICAL_FROM_ISR();

        HAL_ETH_IRQHandler(&heth);

        /* 退出临界段 */
        taskEXIT_CRITICAL_FROM_ISR( ulReturn );
    }


    extern xSemaphoreHandle s_xSemaphore;
    void HAL_ETH_RxCpltCallback(ETH_HandleTypeDef *heth)
    {
    //  LED2_TOGGLE;
        portBASE_TYPE xHigherPriorityTaskWoken = pdFALSE;
        xSemaphoreGiveFromISR( s_xSemaphore, &xHigherPriorityTaskWoken );
        portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
    }



此外我们还需要在网卡初始化的时候创建网卡接收线程与对应的信号量，网卡初始化函数还是low_level_init()函数，
并且初始化的其他信息都未修改，只是添加了线程与信号量的创建，具体见 代码清单:ETH-21_ 。


.. code-block:: c
    :caption: 代码清单:ETH-21 
    :name: 代码清单:ETH-21
    :linenos:

    static void low_level_init(struct netif *netif)
    {
        HAL_StatusTypeDef hal_eth_init_status;

        //初始化bsp—eth
        hal_eth_init_status = Bsp_Eth_Init();

        if (hal_eth_init_status == HAL_OK)
        {
            /* Set netif link flag */
            netif->flags |= NETIF_FLAG_LINK_UP;
        }

    #if LWIP_ARP || LWIP_ETHERNET

        /* set MAC hardware address length */
        netif->hwaddr_len = ETH_HWADDR_LEN;

        /* set MAC hardware address */
        netif->hwaddr[0] =  heth.Init.MACAddr[0];
        netif->hwaddr[1] =  heth.Init.MACAddr[1];
        netif->hwaddr[2] =  heth.Init.MACAddr[2];
        netif->hwaddr[3] =  heth.Init.MACAddr[3];
        netif->hwaddr[4] =  heth.Init.MACAddr[4];
        netif->hwaddr[5] =  heth.Init.MACAddr[5];

        /* maximum transfer unit */
        netif->mtu = NETIF_MTU;

        /* Accept broadcast address and ARP traffic */
        /* don't set NETIF_FLAG_ETHARP if this device is not an ethernet one 

    #if LWIP_ARP
        netif->flags |= NETIF_FLAG_BROADCAST | NETIF_FLAG_ETHARP;
    #else
        netif->flags |= NETIF_FLAG_BROADCAST;
    #endif /* LWIP_ARP */

        /* USER CODE BEGIN PHY_PRE_CONFIG */

        s_xSemaphore = xSemaphoreCreateCounting(40,0);

        /* create the task that handles the ETH_MAC */
        sys_thread_new("ETHIN",
                    ethernetif_input,  /* 任务入口函数 */
                    netif,           /* 任务入口函数参数 */
                    NETIF_IN_TASK_STACK_SIZE,/* 任务栈大小 */
                    NETIF_IN_TASK_PRIORITY); /* 任务的优先级 */

    #endif /* LWIP_ARP || LWIP_ETHERNET */

        /* Enable MAC and DMA transmission and reception */
        HAL_ETH_Start(&heth);
    }

协议栈初始化
^^^^^^^^^^^^^^^^^

经过上面的移植，我们的底层与操作系统接口都基本移植完毕，想要让LwIP在操作系统中能跑起来，还需要最后一步，将协议栈进行初始化，前面我们也说了，
内核在操作系统中是作为一个线程独立存在的，在初始化的时候，我们不仅要挂载网卡，也要创建tcpip_thread()线程，当然，这个线程LwIP会在初始的时候自动创建，
而挂载网卡的内容与无操作系统是一样的，就无需过多修改，协议栈初始化的源码具体见 代码清单:ETH-22_。



.. code-block:: c
    :caption: 代码清单:ETH-22
    :name: 代码清单:ETH-22
    :linenos:

    /*Static IP ADDRESS: IP_ADDR0.IP_ADDR1.IP_ADDR2.IP_ADDR3 */
    #define IP_ADDR0                    192
    #define IP_ADDR1                    168
    #define IP_ADDR2                      1
    #define IP_ADDR3                    122

    /*NETMASK*/
    #define NETMASK_ADDR0               255
    #define NETMASK_ADDR1               255
    #define NETMASK_ADDR2               255
    #define NETMASK_ADDR3                 0

    /*Gateway Address*/
    #define GW_ADDR0                    192
    #define GW_ADDR1                    168
    #define GW_ADDR2                      1
    #define GW_ADDR3                      1
    /* USER CODE END 0 */

    struct netif gnetif;
    ip4_addr_t ipaddr;
    ip4_addr_t netmask;
    ip4_addr_t gw;
    uint8_t IP_ADDRESS[4];
    uint8_t NETMASK_ADDRESS[4];
    uint8_t GATEWAY_ADDRESS[4];

    void LwIP_Init(void)
    {

        tcpip_init(NULL, NULL);

        /* IP addresses initialization */
        /* USER CODE BEGIN 0 */
    #ifdef USE_DHCP
        ip_addr_set_zero_ip4(&ipaddr);
        ip_addr_set_zero_ip4(&netmask);
        ip_addr_set_zero_ip4(&gw);
    #else
        IP4_ADDR(&ipaddr,IP_ADDR0,IP_ADDR1,IP_ADDR2,IP_ADDR3);
        IP4_ADDR(&netmask,NETMASK_ADDR0,NETMASK_ADDR1,NETMASK_ADDR2,NETMASK_ADDR3);
        IP4_ADDR(&gw,GW_ADDR0,GW_ADDR1,GW_ADDR2,GW_ADDR3);
    #endif /* USE_DHCP */
        /* USER CODE END 0 */
        /* Initilialize the LwIP stack without RTOS */
        /* add the network interface (IPv4/IPv6) without RTOS */
        netif_add(&gnetif, &ipaddr, &netmask, &gw, NULL,
                &ethernetif_init, &tcpip_input);

        /* Registers the default network interface */
        netif_set_default(&gnetif);

        if (netif_is_link_up(&gnetif))
        {
        /* When the netif is fully configured this function must be called */
            netif_set_up(&gnetif);
        }
        else
        {
            /* When the netif link is down this function must be called */
            netif_set_down(&gnetif);
        }

    }

在tcpip_init()函数中，LwIP会调用lwip_init()将内核进行初始化，并且创建一个tcpip_mbox邮箱，邮箱的大小是TCPIP_MBOX_SIZE，用于接收从底层或者上层传递过来的消息，
并且最重要的是创建一个tcpip_thread线程，这就是LwIP在操作系统中作为一个独立的线程运行，所有处理的数据都要这个线程去处理，这个线程我们会在后续讲解。lwip_init()函数的源码具体见 代码清单:ETH-23_。



.. code-block:: c
    :caption: 代码清单:ETH-23 
    :name: 代码清单:ETH-23
    :linenos:

    void
    tcpip_init(tcpip_init_done_fn initfunc, void *arg)
    {
        lwip_init();

        tcpip_init_done = initfunc;
        tcpip_init_done_arg = arg;
        if (sys_mbox_new(&tcpip_mbox, TCPIP_MBOX_SIZE) != ERR_OK)
        {
            LWIP_ASSERT("failed to create tcpip_thread mbox", 0);
        }
    #if LWIP_TCPIP_CORE_LOCKING
        if (sys_mutex_new(&lock_tcpip_core) != ERR_OK)
        {
            LWIP_ASSERT("failed to create lock_tcpip_core", 0);
        }
    #endif /* LWIP_TCPIP_CORE_LOCKING */

        sys_thread_new(TCPIP_THREAD_NAME, tcpip_thread, NULL,
                    TCPIP_THREAD_STACKSIZE, TCPIP_THREAD_PRIO);
    }


对于 代码清单:ETH-23_ 中netif_add()函数，与裸机移植的时候有一点细微的差别，它最后
一个参数就是tcpip_input，是一个函数，而不是原来的ethernet_input()，因为tcpip_input()
函数会将网卡收到的数据包打包成为一个消息，发送到tcpip_mbox邮箱中，传递给
tcpip_thread线程去处理，不过本质上也是调用ethernet_input()函数去递交这个数据包，只
不过是绕了一大圈，因此，想要LwIP能正常运行，消息机制是不可或缺的，关于这个运
作机制我们在<LWIP开发实战指南>详细讲解。



移植后使用ping测试基本响应
^^^^^^^^^^^^^^^^^^^^^^^^^^^^


在移植完操作系统与LwIP，我们也应该用电脑去ping一下开发版，看看是否能ping通，如果通了就表明我们的移植是正常的，那么我们才能继续去用LwIP完成更高级的应用,
链路正常时DOC窗口截图如图 ping命令窗口_ 。

.. image:: media/ETH020.png
    :align: center
    :name: ping命令窗口
    :alt: ping命令窗口


