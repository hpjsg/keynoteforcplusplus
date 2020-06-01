* TCP 和 UDP的区别：
1、TCP是面向连接的发送数据之前需要建立连接，UDP是无连接的。
2、TCP是面向字节流的，UDP是面向报文的。
3、TCP提供了可靠性，基于确认重传机制。UDP缺乏可靠性。
4、TCP通过序列号保证了数据的顺序，UDP数据包可能被网络重排序。
5、TCP提供流量控制（通知对端本端接收窗口的大小），UDP不提供流量控制。
6、TCP头部开销大，UDP头部开销小

* ISO 七层模型：
应用层/表示层/会话层/传输层/网络层/数据链路层/物理层  
数据链路层：数据链路层把网络层传下来的分组封装成帧。源自网络层来的数据可靠地传输到相邻节点的目标机网络层。
五层模型：
应用层/传输层/网络层/数据链路层/物理层
四层模型：TCP/IP  分层模型
应用层/传输层/网间层/网络接口层

* 三次握手
第一次握手：建立连接时，客户端发送SYN包到（SYN=J）到服务器，并进入SYN_SEND状态，等待服务器确认。
第二次握手：服务器收到SYN包，必须确认客户的SYN（ACK=J+1），同时自己也发一个SYN包（SYN=K），即ACK+SYN包，服务器进入SYN_RECV状态。
第三次握手：客户端收到服务器的ACK+SYN包，向服务器发送确认包ACK（ACK=k+1）。客户端和服务器端进入ESTABLISHED状态，完成三次握手。
SYN用作以后通信的序号，以保证应用层接收到的数据不会因为网络上的传输问题而乱序。（TCP会用这个序号拼接数据）。

TCP有一个特别的概念叫做半关闭，因为TCP连接是全双工的，因此在关闭连接时，必须关闭发送和接收两个方向上的连接。客户机给服务器发送一个FIN的TCP报文，然后服务器给客户端回一个确认ACK报文，并且再发送一个FIN报文，当客户端回复ACK后，连接就结束了。也叫做4次挥手。

TCP流转图
CLOSED：初始状态
LISTEN:服务器的某个socket处于监听状态，可以接受连接。
SYN_SENT:服务器监听以后，客户端connect连接命令发SYN报文，客户端进入SYN_SENT状态，等待服务器的确认。

SYN_RCVD:表示服务器收到了SYN报文，在正常情况下，这个状态是服务器端的socket在建立TCP连接时的3次握手会话过程中的一个短暂的中间状态。当收到客户端的ACK报文进入ESTABLISHED状态。(客户端收到对ACK+SYN包进入ESTABLISHED)
FIN_WAIT1:建立连接以后，一方想主动终止连接，向对方发送了FIN报文，此时立即进入FIN_WAIT1状态。当对方回应ACK报文到达后，立刻进入FIN_WAIT_2状态。
FIN_WAIT_2状态：一方请求关闭连接，不在发送数据，但是仍可以接受对方的数据。
TIME_WAIT: 主动关闭方表示收到了对方的FIN报文，并发送了ACK报文，就等2MSL后可进入CLOSED可用状态。如果FIN_WAIT_1状态下，收到了对方同时带FIN和ACK标志的报文，可直接进入TIME_WAIT状态，无需经过FIN_WAIT_2状态。（四次挥手变三次）
这个状态很大程度上保证了双方都可以正常的结束，但是由于2MSL的等待状态，应用程序在这一段时间内无法使用同一个socket

CLOSE_WAIT:等待关闭状态。对方发送了FIN报文，返回ACK给对方后，此时进入CLOSE_WAIT状态。等待应用层关闭连接，发送FIN报文给对方。
LAST_ACK：被动关闭方等待最后的ACK报文。
RST：同时打开和同时关闭，特殊的TCP状态。
 
半关闭：主动关闭方发送了FIN报文，被动关闭方接收到然后发送了ACK。在被动关闭方发送FIN报文之前，双方处于半关闭状态。数据只有可能从被动关闭一端流向主动关闭一端（但是主动关闭一方是不能read的，close将一个TCP套接字标记为已关闭，然后该套接字描述符不能再由调用进程使用，如果想读要使用shutdown命令）。
半连接：在三次握手中，服务器发送了ACK和SYN后没有收到客户端的第三次握手。半连接攻击，消耗服务器的内存资源（发送SYN但是不进行第三次握手，SYN报文装填端口的未完成队列，使得合法SYN无法排队，SYN泛洪攻击）。
半打开：一方已经关闭或者异常关闭，另一方还不知道，称之为半打开。可以引入心跳机制。

MAC地址与IP：
MAC: 用于物理层和数据链路层。我们的通讯设备，无论是手机、电脑或者其他设备，都会有一个独一无二、固定的mac地址。mac地址与我们的设备进行绑定，就能确定我们身份。其实MAC地址，并不能算是地址，更应该算是一个身份证，用来表明身份。
IP: 用于网络层。IP协议提供的一种统一的地址格式，它为互联网上的每一个网络和每一台主机分配一个逻辑地址，以此来屏蔽物理地址的差异。

IP地址是变化的，是基于网络拓扑结构的，用于屏蔽物理链路的差异。MAC地址是不变的，是基于设备的，表明设备的身份。
有MAC为什么还要IP？
1、只拥有MAC地址的话，只有在同一网络区域内，才能进行数据传输，不能跨网络区域。（不同局域网技术很多）
2、如果想跨网络区域进行数据传递，最现实的方法就是借助ISP提供的网络区域。
3、ISP能提供全球互联的网络——因特网，借助因特网可以传输数据给连接因特网上的机器。

设备越来越多，路由变得困难。将网络分成若干子网。路由时，路由器把其他子网作为一个整体。对于目的地在其他子网的数据包，路由器只负责让数据包到达子网即可，剩下的工作由子网内部解决。

只使用mac地址，路由器需要记住每个mac地址所在子网是哪一个，有2^48个mac地址，需要非常大的存储空间。
IP地址和网络拓扑结构相关，对于同一个子网上的设备，分配给它们的ip地址的前缀想同。路由器通过ip地址的前缀就能知道设备在哪一个子网上，路由器只需记住每个子网的位置，大大减少了路由器所需内存。

IP地址要设备上线后，才能根据设备进入子网来动态分配唯一的ip，需要根据mac地址，来判断ip上的设备是否发生变化，是否是目的设备。


地址解析协议ARP。实现由IP地址得到MAC地址。 

数据包路由过程：

1、 数据包发送目标地址是否在同一子网，在通过交换机直接传送。(ARP找到目标mac，将目的IP和mac写入数据包，由交换机发送到目标)。如果不在同一子网，主机将数据包发送到默认网关。将目标mac地址写为路由器的mac地址。
2、路由器接到帧，路由器在路由表中发现目标网络，选择一个更接近目标的下一跳，下一跳的MAC地址被确定
3、 路由器接到帧，路由器发现目的ip地址在它的直连网络内，最终主机的MAC地址被确定，帧中的数据包被发送到最终主机。
            
在数据包端到端的传输过程中，ip地址始终不会发生改变，而MAC地址则随着具体链路的不同而不同(源：上一跳，目的：下一跳)。
路由器在某一个入接口上接收到数据帧后，先检测目的地是否是自己。若是，则交给上层处理，否则会缓存数据包内容，然后根据目标地址查找路由表找到相关表项，得到NEXT HOP及出接口的MAC地址，用这两个地址作为新的目的及源MAC地址封装事先缓存的数据包，然后转发，这个过程称为帧的重写（REWRITE）。


DDOS攻击：Distributed Denial of Service分布式拒绝服务。DOS：拒绝服务，主要思想是使某个服务不可用。
ICMP泛洪攻击/UDP泛洪攻击(UDP协议的无连接性，方便伪造源地址)/SYN泛洪攻击/NTP泛洪攻击
发送NTP请求给开放的NTP服务器，源IP地址被伪造成攻击目标的IP。NTP服务器会将NTP响应发送到目标IP。(NTP是标准的基于UDP协议传输的网络时间同步协议)

防护：
1、黑名单
2、DDOS清洗。对用户请求数据进行实时监控，及时发现DOS攻击等异常流量，清洗掉这些异常流量。


SYN泛洪攻击的解决办法：
1、缩短SYN-RCVD时间
2、增加TCP监听套接字的backlog队列
3、基于IP地址的过滤

4. time_wait 
存在理由：
1.可靠的实现TCP全双工连接的终止。最终的ACK丢失，对方会重发一个FIN报文，如果不处于TIME_WAIT维护状态信息，就不能重新发ACK
2.允许老的重复分节在网络中消逝。防止老的分节对新的连接实体造成干扰。

TCP选项SO_REUSEADDR:用于对TCP套接字处TIME_WAIT状态下的socket，可以立即被重复使用绑定。重启监听服务器。
1）SO_REUSEADDR允许启动一个监听服务器并捆绑众所周知端口，即使以前建立的以该端口用作它们的本地端口的连接仍存在。
2）SO_REUSEADDR允许在同一端口上启动同一服务器的多个实例，只要每个实例捆绑一个不同的本地IP地址即可
3）SO_REUSEADDR允许单个进程捆绑同一个端口到多个套接字上，只要每次捆绑指定不同的本地IP地址即可
4）SO_REUSEADDR允许完全重复的捆绑：一般本特性仅支持UDP套接字。

SO_REUSEPORT:不能解决TIME_WAIT需要每次绑定socket时都声明这个选项。 但可以启动多个服务器进程，监听socket绑定到相同ip和port。Linux内核会做简单的负载均衡，选择一个进程accept返回。这样可以提高服务器的负载。
1）本选项允许完全重复的捆绑，不过只有在想要捆绑同一IP地址和端口的每个套接字都指定了本套接字选项才行。
2）如果被捆绑的IP地址是一个多播地址，那么SO_REUSEADDR和SO_REUSEPORT被认为是等效的

简单治标方案：修改内核参数 /proc/sys
net.ipv4.tcp_tw_reuse = 1
#表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
net.ipv4.tcp_tw_recycle = 1
#表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。
系统改进方案：
1、增加更多的四元组数目，比如，服务器可用端口，或服务器IP。让服务器能容纳足够多的TIME-WAIT状态连接。修改net.ipv4.ip_local_port_range参数，增加客户端端口可用范围。/增加服务端端口，多监听一些端口/尤其是作为负载均衡服务器时，使用更多IP去跟后端的web服务器通讯/增加服务端IP。
2、服务器尽量不主动关闭连接。
3、服务器发起连接时，使用长连接，不要用短连接，可减少TIME_WAIT状态tcp连接(服务器连接数据库，需要服务器主动断开)。

SO_LINGER 默认close立即返回，系统会尝试将发送缓冲区中的剩余数据发送给对端。SO_LINGER可以改变close行为。
l_onoff非0，l_linger为0，close不阻塞丢弃发送缓冲区数据，向对端发送RST报文。强制关闭。
l_linger非0，close返回会延迟。进程阻塞直到send buffer中的数据发送完并受到对方应答消息/或者延迟时间消耗完，将send buffer中数据丢弃。 socket是非阻塞时，程序不会等待，丢弃所有数据。 


Close_wait 占据大量端口
可能原因：服务端接口的操作耗时较长，客户端主动断开了连接，此时，服务端就会出现 close_wait。尤其是服务器有长期阻塞的操作，没法响应客户端的FIN报文。
1.服务器访问数据库等应避免阻塞，设置超时时间等。
2.当客户端因为某种原因先于服务端发出了FIN信号，就会导致服务端被动关闭，若服务端不主动关闭socket发FIN给Client，此时服务端Socket会处于CLOSE_WAIT状态（而不是LAST_ACK状态）。通常来说，一个CLOSE_WAIT会维持至少2个小时的时间（系统默认超时时间的是7200秒，也就是2小时）。如果服务端程序因某个原因导致系统造成一堆CLOSE_WAIT消耗资源，那么通常是等不到释放那一刻，系统就已崩溃。因此，解决这个问题的方法还可以通过修改TCP/IP的参数来缩短这个时间，于是修改tcp_keepalive_*系列参数（keep alive是服务器用于探测客户端的）。

TCP滑动窗口：
滑动窗口的两个作用：1、提供tcp的可靠性2、提供tcp的流控特性
TCP的一个窗口是一个16bit位字段，代表窗口容量，TCP标准窗口最大为65535个字节。TCP 是一个全双工的协议，所以窗口分为发送窗口和接收窗口。接收窗口会告诉对方本端的TCP缓冲区还能接受多少字节的数据来控制对方的发送速度。
发送窗口是指 已发送但未收到ACK及未发送但对端允许发送的数据。
接受端缓存中有3种状态：1.已接受2.未接受但准备接受3.未接受但不准备接受。其中未接受但准备接称成为接收窗口。
滑动窗口实现面向流的可靠性基于确认重传机制。发送窗口只有收到对端对于发送窗口内字节的ACK确认才会移动发送窗口的左边界。接收窗口只有在前面所有的段都接收的情况下才会移动左边界。在前面还有字节未接收但收到后面字节的情况下，窗口不会移动，并不对后续字节确认，以此确保对端对这些数据重传。

TCP重排序细节：
如果乱序报文在接收窗口内，TCP不会丢弃它，将其保存在连接的重组队列中，等待中间缺失的报文段。乱序报文段使用双向链表来组织。
1.报文段到达次序正确2.重组队列为空3.连接处于ESTABLISHED状态
以上三个条件有一个不满足报文段放入重组队列。（关于3，SYN报文段可以携带数据，但是在连接进入ESTABLISHED之前保存在重组队列中，不可提交应用程序）。
TCP_REASS函数将乱序报文段放入重组队列。
从链表的头部遍历，找到序号大于接收报文段ti的序号的第一个报文段q，如果双向链表中q之前还有报文段p，要判断p是否与ti有重复字节数，去重。判断ti和q之间是否有重复字节数，去重。然后将ti插入链表中。如果可能向进程递交数据。（第一个报文段的序号等于等待接收的下一序号就可以将数据向上递交，直到存在缺失报文段）。

单工通信：单向传输 半双工通信：双向交替传输 全双工通信：双向同时传输 


ICMP(网际控制报文协议)
ICMP是为了更有效地转发IP数据报和提高交付成功的机会。它封装在IP数据报中，但是不属于高层协议。
1、Ping 是ICMP的一个重要应用，主要用来测试两台主机之间的连通性。 Ping的原理是通过向目的主机发送ICMP Echo请求报文，目的主机收到之后会发送Echo回答报文。Ping会根据时间和成功响应的次数估算出数据包往返时间以及丢包率。
2、Traceroute是ICMP的另一个应用，用来跟踪一个分组从源点到终点的路径。Traceroute发送的IP数据报封装的是无法交付的UDP用户数据报，并由目的主机发送终点不可达差错报告报文。


connect 函数将激发TCP的三次握手过程，仅在连接建立成功或者出错才返回。有以下错误：
1.没有收到SYN的ACK分节，75秒后会返回ETIMEDOUT错误。
2.若对客户的响应是RST（表示复位）表明服务器在指定端口上没有进程在等待连接（没有listen的服务器）。客户端会收到RST会立马返回ECONNREFUSED错误。

Listen 函数：
1、调用listen将套接字从主动套接字转换为被动套接字，指示内核接受指向该套接字的连接请求。套接字从closed状态转为listen状态。
2、函数的backlog参数指定了内核为相应套接字 排队的最大连接个数。
内核为任何一个监听套接字维护两个队列：
1、未完成连接队列，半连接状态的套接字，处于SYN_RCVD状态(保留75s，有超时处理)。
2、已完成连接的队列。套接字处于ESTABLISHED状态。
服务器收到客户端的ACK后会将套接字从未完成连接的队列转移至已完成连接的队列。
在accept之前到达的数据会放入socket的缓冲区中
如果客户SYN到达，而队列是满的，TCP就忽略该分节，因为这种情况只会短暂出现，客户端将会重发SYN。

Accept函数用于从已完成连接队列的头部返回一个已完成连接，如果已完成连接队列为空，进程阻塞。

封包和解包：
粘包： 
1、由Nagle算法造成的发送端的粘包。Nagle算法是一种改善网络传输效率的算法。当要提交一段数据给TCP发送时，TCP不会立即发送，而是等待一段时间，看在等待期间是否还有要发送的数据，若有则一次将多段数据一起发送出去。
2、接收端接受不及时造成的接收端粘包。TCP会把接收到的数据存到缓存区中然后通知应用层取数据。当应用层没有及时取出数据时，会造成TCP缓存区存放了多段数据。

封包与拆包
封包给一段数据加上包头，包头是个固定长度的结构体，其中有个字段用来表示包体的长度。接收端根据固定的包头长度和包头中的包体长度正确拆分出一个完整的数据包。

* TCP流量控制：
如果发送端发送数据太快接收端来不及接受会造成数据丢失。流量控制是通过大小可变的滑动窗口实现的。接受端在返回ACK给发送端时，通过TCP头部的TCP窗口大小字段（16位）和和窗口扩大因子字段告诉发送端接收窗口的大小。如果接收缓冲区满，接收端会将窗口设为0，这时发送端不发送数据，但要定期发送窗口探测数据段，来获得（ACK中的）接收窗口大小（以防有窗口通告丢失，因为窗口通告（ACK）不会被确认，是不可靠的）。

* TCP拥塞控制
拥塞控制是防止过多数据注入网络中，导致路由器和链路过载。拥塞控制是一个全局性的过程，流量控制是点对点通信量的控制。4个核心算法组成：慢启动/拥塞避免/快重传/快恢复
慢启动：发送方维持一个拥塞窗口(cwnd congestion window，发送窗口小于接收窗口和拥塞窗口)状态变量。TCP连接初始化，拥塞窗口设为1个MSS大小。然后执行慢启动，拥塞窗口按指数规律增长（每收到一个报文段确认，窗口就增加至多一个MSS，收到单个确认但一次确认确认多个数据报窗口增加相应数值，所以一次成功传输之后拥塞窗口加倍）直到窗口大小等于慢启动的门限，开始执行拥塞避免算法。
拥塞避免：拥塞避免让拥塞窗口成线性规律增长，每经过一个往返时间RTT就把发送方的拥塞窗口cwnd加一，而不是加倍。
拥塞窗口大小等于慢启动门限时，两个算法都可以使用。当发送端判断网络发生拥塞时，不论出于慢启动还是拥塞控制阶段，都要将慢启动的门限更新为拥塞情况出现时发送窗口大小的一半，但不能小于2，然后将拥塞窗口重新设置为1个MSS，重新开始执行慢启动。发送端判断拥塞的依据（传送超时未收到ACK报文，TCP重传定时器溢出）
快重传：接收端每当收到一个失序的报文段后立即发出重复的确认（使发送端及早知道有报文段没有到达对方）而不是等待自己发送数据时捎带确认。发送方只要一连收到3个重复确认就应当立即重传对方尚未收到的报文段，而不必等待报文的重传计时器到时。
快恢复：发送方连续收到3个重复确认时，执行乘法减小算法，把慢启动的门限减为当前发送窗口的一半，将拥塞窗口大小设为慢启动的门限，开始执行拥塞避免算法，加法增大。

TCP 定时器：
1、重传定时器：定时器超时未收到对报文段的确认，重传报文段，定时器复位。
2、坚持计时器：发送端收到一个窗口大小为零的确认时，就启动坚持计时器。当坚持计时器到时，发送端就发送一个探测报文段。(因为窗口通告是不可靠的，用于获得窗口通告)
3、keep-alive计时器：探测客户端是否退出、异常退出。
4、time-wait计时器：2MSL


最大并发连接数限制   ** https://blog.csdn.net/Solstice/article/details/5950190
服务器进程文件描述符创建达到上限，如何应对新到的请求？
accept 返回EMFILE错误 显示 too many open files。进程文件描述符用完，无法创建socket fd，由于没有拿到socketfd，无法close这个连接。下一次epoll_wait()时候立刻返回，因为新的连接还在那里，listenfd还是可读的。程序会陷入busy loop，cpu占用率高。
调高 ulimit -n 修改当前进程打开文件描述符的数量限制 /etc/security/limits.conf 永久修改   /proc/sys/fs/file-max是所有进程最大的文件数，方法不适合对外的连接服务器。
退出 暂时的错误，不值得退出
关闭listenfd，不知道什么时候可以重开。
Libevent
准备一个空闲的文件描述符，遇到这种情况，先关闭这个空闲文件，获得一个文件描述符名额。再accept拿到新的socket连接的描述符。随后close它，优雅的断开客户端连接。最后重新打开一个空闲文件，把“坑”站住，已备下一次使用。多线程下有race condition。缺点是：关于这个fd的操作不是线程安全的。

在程序中限制连接并发数，不让进程的文件描述符耗光。用一个变量统计连接数。超出限制后，服务器主动关闭连接。可以保证服务器不会过载。


select的timeout 参数精度为1ns，而poll和epoll为1ms，因此select更加适用于实时性要求比较高的场景，比 如核反应堆的控制。
select 可移植性更好，几乎被所有主流平台所支持。
poll应用场景 
poll使用链表保存文件描述符，因此没有了监视文件数量的限制，如果平台支持并且对实时性要求不高，应该使用poll而不是select
只需要运行在Linux平台上，有大量的描述符需要同时轮询，并且这些连接好是长连接。 
需要监控的文件描述符状态变化多，而且都是非常短暂的，也没有必要使用 epoll。因为epoll中的所有描述符都存储在内核中，造成每次需要对描述符的状态改变都需要通过 epoll_ctl() 进行系统调用，频繁系统调用降低效率。并且epoll的描述符存储在内核，不容易调试。 

Epoll
EPOLL相对于select和poll的优势：
epoll为什么高效，相比select和poll
1、select 最让人不能忍受的是一个进程所打开的fd是有一定限制的，由FD_SETSIZE设置(select本身的设置)，默认值是1024。对于那些需要支持的上万连接数目的服务器来说显然太少了。epoll没有这个限制，它所支持的fd的上限是最大可以打开的文件的数目，这个数字远大于2048,1GB内存中是10万左右。

2、select/poll的一个致命弱点是，当你有一个很大的socket集合时，由于网络的延迟，任意时间只有部分socket是活跃的。但select/poll在内核中是轮询的，线性扫描全部集合，导致效率下降，而epoll只响应活跃的socket。Epoll维护一个rdllist链表用于存放就绪的事件。执行epoll_ctl时，除了把fd对应事件放到epoll在内核中建的红黑树上之外，还会给内核中断处理程序注册一个回调函数，如果这个fd的中断到了，就把它放到rdllist链表里。当一个fd（例如socket）上有数据到了，内核在把设备（例如网卡）上的数据copy到内核中后就来把fd（socket）插入到准备就绪rdllist里了。所以epoll不存在这个问题，它只响应活跃的socket，因为在内核中epoll是根据每个fd上的回调函数实现的。当socket活跃时调用回调函数，其他状态下socket不会。

3、无论是select、poll还是epoll都需要内核把文件描述符的消息通知给用户空间，如何避免不必要的内存拷贝非常重要。
对于select/poll而言每次调用都要传递所要监控的所有fd给select/poll系统调用（这意味着每次调用都要将fd列表从用户态拷贝到内核态，当fd数目很多时，这会造成低效）。当事件发生后，又将数据拷贝到用户空间。
* 而且select返回整个句柄数组，应用程序需遍历才能发现哪些句柄发生了事件。而每次调用epoll_wait时，不需要再传递fd列表给内核，在epoll_ctl中将需要监控的fd告诉了内核（epoll_ctl不需要每次都拷贝所有的fd，只需要进行增量式操作）。返回时epoll_wait每次只返回就绪的fd集合，仅需要从内核态copy少量的fd到用户态。


epoll从O(n)优化到O(1)怎么做的？
红黑树组织epitem结构体 + 内核中断处理程序注册回调函数 + 就绪列表rdllist 这里O(1)指的是采用回调方式检测就绪事件的时间复杂度和应用程序索引就绪文件描述符的复杂度。O(n)对应的是轮询方式检测就绪事件，应用程序用轮询方式检测就绪文件描述符。

Epoll 主要过程
为每一个epoll_event(fd/events)创建关联的epitem结构体。用红黑树组织所有添加到epoll上的事件，双向链表rdllist保存着将要通过epoll_wait返回用户的事件。epoll_ctl添加事件时除了将epitem结构体插入红黑树，还会给内核中断处理程序注册一个回调函数，如果这个fd的状态改变(中断到了)，就把它对应的epitem放到rdllist链表里。epoll_wait检查rdllist双向链表是否有epitem元素，如果有拷贝到txlist，然后清空rdlist。接着扫描txlist中的每个epitem，调用其关联fd对应的poll方法(fd对应file_struct的files_operation里面的poll接口，用于获取当前的文件状态)。此时对poll的调用仅仅是取得fd上较新的events（防止之前events被更新），之后将取得的events和相应的fd发送到用户空间（封装在struct epoll_event，从epoll_wait返回）

为每一个epoll_event(fd/events)创建关联的epitem结构体。用红黑树组织所有添加到epoll上的事件，双向链表rdllist保存着将要通过epoll_wait返回用户的事件。epoll_wait返回的条件是rdlist不空。
之后如果这个epitem对应的fd是LT模式监听且取得的events是用户所关心的(LT模式下只有poll到0才不再加入rdllist)，不管它还有没有有效的事件或者数据, 都会被重新插入到ready list中，在下一次epoll_wait 时, 会立即返回, 并通知给用户空间. 当然如果这个被监听的fd确实没事件也没数据了, epoll_wait会空转一次。如果是ET, epitem是不会再进入到readly list, 除非fd再次发生了状态改变, fd上注册的回调函数被调用. 

关于状态改变：
对于读取操作： 
(1) 当buffer由不可读状态变为可读的时候，即由空变为不空的时候。 
(2) 当有新数据到达时，即buffer中的待读内容变多的时候。 
对于写操作： 
(1) 当buffer由不可写变为可写的时候，即由满状态变为不满状态的时候。 
(2) 当有旧数据被发送走时，即buffer中待写的内容变少得时候。

Epoll ET 模式
1.使用非阻塞文件描述符（没有数据立刻返回EAGAIN错误）;
2.read(2)或write(2)直到EAGAIN才退出。

EPOLL 水平触发：来自内核的就绪通知只是一个提示；当试图读取文件描述符时，它可能不再就绪。这就是为什么在使用就绪通知时使用非阻塞模式很重要的原因。

ET vs LT
LT模式是epoll默认的工作方式，相当于一个效率很高的poll模型；而ET是高效的工作方式。