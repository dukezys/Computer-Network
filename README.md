# Computer-Network
   * [Computer-Network](#Computer-Network)
      * [运输层](#运输层)
         * [运输层协议概述](#运输层协议概述)
            * [进程之间的通信](#进程之间的通信)
            * [运输层的两个主要协议](#运输层的两个主要协议)
            * [运输层的端口](#运输层的端口)
         * [用户数据报协议UDP](#用户数据报协议udp)
            * [UDP概述](#udp概述)
            * [UDP的首部格式](#udp的首部格式)
         * [传输控制协议TCP](#传输控制协议tcp)
            * [TCP主要特点](#tcp主要特点)
            * [TCP的连接](#tcp的连接)
            * [TCP报文段的首部格式](#tcp报文段的首部格式)
            * [可靠传输的工作原理](#可靠传输的工作原理)
               * [停止等待协议](#停止等待协议)
               * [连续ARQ协议](#连续arq协议)
            * [TCP可靠传输的实现](#tcp可靠传输的实现)
               * [滑动窗口](#滑动窗口)
               * [超时重传时间的选择](#超时重传时间的选择)
            * [TCP的流量控制](#tcp的流量控制)
               * [利用滑动窗口实现流量控制](#利用滑动窗口实现流量控制)
               * [TCP的传输效率](#tcp的传输效率)
            * [TCP的拥塞控制](#tcp的拥塞控制)
               * [拥塞控制的一般原理](#拥塞控制的一般原理)
               * [TCP的拥塞控制方法](#tcp的拥塞控制方法)
                  * [慢开始](#慢开始)
                  * [拥塞避免算法](#拥塞避免算法)
            * [TCP的运输连接管理](#tcp的运输连接管理)
               * [TCP的连接建立](#tcp的连接建立)
               * [TCP的连接释放](#tcp的连接释放)

## 运输层
### 运输层协议概述
#### 进程之间的通信

* 运输层向它上面的应用层提供通信服务，它属于面向通信部分的最高层，同时也是用户功能中的最底层。
* 两台主机进行通信就是两台主机中的应用进程互相通信。从运输层的角度看，通信的真正端点并不是主机而是主机中的进程，端到端的通信是应用进程之间的通信。
* **网络层为主机之间提供逻辑通信，而运输层为应用进程之间提供端到端的逻辑通信**
* 运输层向高层用户屏蔽了下面网络核心的细节，它使应用进程看见的就是好像在两个运输层实体之间有一条端到端的逻辑通信信道。
#### 运输层的两个主要协议
* TCP/IP运输层的两个主要协议都是互联网的正式标准：**用户数据报协议UDP(User Datagram Protocol)和传输控制协议(Transmission Control Protocol)**
* UDP在传送数据之前不需要先建立连接。远地主机运输层收到UDP报文后，不需要给出任何确认。虽然UDP不提供可靠交付，但在某些情况下UDP却是一种最有效的工作方式。
* TCP则提供面向连接的服务。在传送数据之前必须先建立连接，数据传送结束后释放连接，增加许多的开销。
#### 运输层的端口
### 用户数据报协议UDP
#### UDP概述

* UDP是无连接的。
* UDP尽最大努力交付。
* UDP是面向报文的。UDP对应用程序交下来的报文，在添加首部后就向下交付IP层，既不合并也不拆分，保留这些报文的边界。
* UDP没有拥塞控制。因此网络出现的拥塞不会使源主机的发送速率降低，对某些实时应用是很重要的，例如IP电话，实时视频会议。
* UDP支持一对一、一对多、多对一和多对多的交互通信。
* UDP首部开销小，只有8个字节，TCP首部为20个字节。
* 没有拥塞控制可能会引发网络产生严重的拥塞问题。
#### UDP的首部格式

* UDP有两个字段：数据字段和首部字段。首部字段只有8个字节，有四个2字节的字段组成。
    1) 源端口：源端口号，在需要对方回信时选用，不需要时可全用0。
    2) 目的端口：目的端口号。在终点交付报文时必须使用。
    3) 长度：UDP用户数据报的长度，最小值是8，仅有首部。
    4) 检验和：检测UDP用户数据报在传输中是否有错，有错丢弃。

* 如果接收方发现目的端口号不正确，就丢弃该报文，并由网际控制报文协议ICMP发送端口不可达差错报文。
* 在计算检验和时，要在UDP之前增加12个字节的伪首部，不向下传递也不递交，只是为了计算检验和。
* **IP数据报的检验和只检验IP数据报的首部，但UDP的检验和是把首部和数据报一起都检验。**
### 传输控制协议TCP
#### TCP主要特点

* TCP是面向连接的运输层协议，传输前建立连接，结束后释放。
* 每一条TCP连接只能是点对点的。
* TCP提供了可靠交付的服务，通过TCP连接传送的数据，无差错、不丢失、不重复，并且按序到达。
* TCP提供全双工通信。允许双方的应用进程在任何时候都能发送数据，TCP连接的两端都设有发送缓存和接收缓存。
* 面向字节流：TCP中的流是流入到进程或从进程流出的字节序列。
虽然应用程序和TCP的交互是一次一个大小不等的数据块，但TCP把应用程序交下来的数据仅仅看成是一连串的无结构的字节流。TCP不知道字节流的含义，不保证接收方锁收到的数据块和发送方发出的数据块具有对应大小的关系。

#### TCP的连接

* 每一条TCP连接有两个端点，称为套接字socket或插口，端口号拼接到concatenated with IP地址构成了套接字，例如192.3.4.5: 80。
* 每一条TCP连接唯一地被通信两端的两个端点确定。
* TCP连接是由协议软件所提供的的一种抽象，TCP连接的端点是个很抽象的套接字，同一个IP地址可以有多个不同的TCP连接，同一个端口号也可以出现在多个不同的TCP接连中。
#### TCP报文段的首部格式

* TCP传送的数据单元是报文段，分为首部和数据两部分。
* TCP报文段首部的前20个字节是固定的，后面有4n字节是根据需要而增加的。
* 首部各字段意义如下：
    1) 源端口和目的端口：各占2字节，分别写入源端口和目的端口号
    2) 序号：占4字节
    3) 确认号：占4字节，是期望收到对方下一个报文段的第一个数据字节的序号
    4) 数据偏移：占4位，指出TCP报文段的数据起始处距离TCP报文段的起始处有多远，实际指出TCP报文段的首部长度。数据偏移的最大值是60字节，即最大首部长度。
    5) 保留：占6位，有6个控制位。
        1) 紧急URG：为1时，紧急指针字段有效。它告诉系统此报文段中有紧急数据，应尽快传送。
        2) 确认ACK：为1时字段有效，0时无效。TCP规定，在连接建立后所有传送的报文段都必须把ACK置为1.
        3) 推送PSH：发送方TCP把PSH置1，并立即创建一个报文段发送出去，接收方TCP收到PSH = 1的报文段，就尽快地交付接收应用进程，而不用等到整个缓存都填满了之后再向上交付。
        4) 复位RST：为1时表明TCP连接中出现严重差错，必须释放连接再重新建立。
        5) 同步SYN：在连接建立时用来同步序号。为1就表示这是一个连接请求或连接接受报文。
        6) 终止FIN：为1时，表示此报文段的发送方的数据已发送完毕，并要求释放运输连接
    6) 窗口：占2字节，值为[0, 2^16 - 1]之间的整数，指的是发送本报文段的一方的**接收窗口**(不是自己的发送窗口)。指出了现在允许对方发送的数据量，值经常变换。 
    7) 检验和：占2字节。检验和字段检验的范围包括首部和数据这两部分。要加12字节的伪首部，但应把第四个字段的17改为6，TCP的协议号是6，把第五字段的UDP长度改为TCP长度。
    8) 紧急指针：在URG = 1时才有意义，指出了本报文段中的紧急数据的字节数。
    9) 选项：长度可变，最长为40字节。

#### 可靠传输的工作原理
##### 停止等待协议

* 无差错情况
* 出现差错：接收方接收时检测到了差错，丢弃，不会发送任何信息。发送端超过一段时间没有收到确认，就认为刚才发送的分组丢失了，重传前面发送过的分组，这时**超时重传**。实现超时重传需要一个**超时计时器**，如果在超时计时器到期之前收到了对方的确认，就撤销已设置的超时计时器。
![](https://github.com/dukezys/Computer-Network/blob/master/pics/StopWait.png)

* **注意三点：**
    1) 发送方发送完一个分组后，必须暂时保留已发送的分组的副本，只有收到相应的确认后才能清除暂时保留的分组副本。
    2) 分组和分组确认都必须进行编号，确认哪个发送出去收到了哪个没收到。
    3) 超时计时器的重传时间应当比数据在分组传输的平均往返时间更长一些。

* 确认丢失和确认迟到：
    1) 接收方发送的对M1的确认丢失了，发送方就要重传M1，接收方又收到了M1，接收方会丢弃这个重复的分组M1，不向上层交付，并向A发送确认
    2) 传输没有出差错，但接收方对分组M1的确认迟到了，发送方会收到重复的确认，丢弃。

* 可靠的传输协议常称为自动重传请求ARQ(Automatic Repeat Request)。
* 信道利用率：往返时间RTT。为了提高传输效率，采用流水线传输，发送方可以连续发送多个分组，不必每发完一个分组就停下来等待确认。
##### 连续ARQ协议
* 将位于发送窗口的分组连续发送出去，而不需要等待对方的确认。按照分组序号大小发送。
* 发送方每收到一个确认，就把窗口向前滑动一个分组的位置。
* 接收方一般采用积累确认的方式，不必对接受的分组逐个发送确认，而是在收到几个分组后对按序到达的最后一个分组发送确认。
* 优点：容易实现，即使确认丢失也不必重传。缺点：不能向发送方反映出接收方已经正确收到的所有分组的信息。
![连续ARQ](https://github.com/dukezys/Computer-Network/blob/master/pics/ARQ.png)


#### TCP可靠传输的实现
##### 滑动窗口
* TCP的滑动窗口是以字节为单位的。凡是已经发送过的数据，在未收到确认之前都必须暂时保留，以便在超时重传时使用。
* 发送窗口的位置由窗口前沿和后沿的位置共同确定。发送窗口后沿的变化情况由两种可能，即不动(没有收到新的确认)和前移(收到了新的确认)。后沿不可能向后退，前沿通常是不断向前移动，但也有可能不移动，也有可能向后收缩但TCP标准不建议这样做。
* 发送缓存用来暂时存放：发送应用程序传送给发送方TCP准备发送的数据；TCP已经发送出但尚未收到确认的数据。发送缓存和发送窗口的后沿是重合的。
* 接收缓存用来暂时存放：按序到达的但尚未被接收应用程序读取的数据；未按序到达的数据。如果收到的分组检测出有错，则要丢弃。如果接收应用程序来不及读取，接收缓存则会被填满，使接收窗口减小到零。反之，接收窗口就可以增大，但不能超过最大缓存。
##### 超时重传时间的选择

* TCP采用了一种自适应算法，它记录一个报文段发出的时间，以及收到相应的确认的时间。时间之差就是报文段的往返时间RTT。RTT的一个加权平均往返时间RTTs。
        
```math
NewRTTs = (1 - a) * (OldRTTs) + a * (NewRTT)
```

* 0 <= a < 1，推荐a = 0.125。
* 超时计时器设置的超时重传时间RTO应略大于上面的RTTs
```math
RTO = RTTs + 4 * RTTd
```

* RTTd是RTT的偏差的加权平均值，B是一个小于1的系数，推荐为0.25
```math
NewRTTd = (1 - B) * (OldRTTd) + B * |RTTs - NewRTT|
```

* **如何判定此确认报文段是对先发送的报文的确认，还是对后来重传的报文段确认？**报文段每重传一次，就把超时重传的时间RTO增大一些。典型的做法是取新的重传时间为旧的重传时间的2倍。当不再发生报文段的重传时，才根据上面的公式计算RTO。
#### TCP的流量控制
##### 利用滑动窗口实现流量控制

* 流量控制：让发送方的发送速率不要太快，要让接收方来得及接收。
* 发送方的发送窗口不能超过接收方给出的接收窗口的数值。**TCP的窗口单位是字节，不是报文段。**
* 为了解决接收方通知发送方有了空闲的存储空间的报文段丢失，TCP为每一个连接设有一个持续计时器，只要连接的一方收到对方的零窗口通知，就启动持续计时器。若持续计时器设置的时间到了，就发送一个零窗口探测报文段，而对方就在确认时给出了现在的窗口值。
##### TCP的传输效率
#### TCP的拥塞控制
##### 拥塞控制的一般原理
* 在某段时间，若对网络中某一资源的需求超过了该资源所能提供的可用部分，网络的性能就会变坏，这种情况就叫拥塞。
##### TCP的拥塞控制方法

* TCP进行拥塞控制的算法有四种：慢开始(slow-start)、拥塞避免(congestion avoidance)、快重传(fast retransmit)、快恢复(fast recovery)。
###### 慢开始

* 拥塞控制也叫基于窗口的拥塞控制。发送方维持一个叫拥塞窗口的cwnd(congestion window)的状态变量，大小取决于网络的拥塞程度，发送方让自己的发送窗口等于拥塞窗口。
* **慢开始算法：当主机开始发送数据时，由于不清楚网络的负荷情况，所以如果立即把大量数据字节注入到网络，那么就有可能引起网络发生拥塞。较好的方法是先探测一下，即由小到大逐渐增大发送窗口，也就是说，由小到大逐渐增大拥塞窗口数值。**
* 慢开始规定，在每收到一个对新的报文段的确认后，可以把拥塞窗口增加最多一个SMSS的数值。每经过一个传输轮次(transmission round)，拥塞窗口cwnd就加倍。开始时cwnd值为1。
* 一个传输轮次所经历的时间其实就是往返时间RTT。
* 在TCP实际的运行中，发送方只要收到一个对新报文段的确认，其拥塞窗口cwnd就立即加1，并可以立即发送新的报文段，而不需要等这个轮次中所有的确认都收到后再发送新的报文段。
###### 拥塞避免算法

* 拥塞避免算法的思路是让拥塞窗口cwnd缓慢地增大。即每经过一个往返时间RTT就把发送方的拥塞窗口cwnd加一个MSS大小。在拥塞避免阶段，拥塞窗口cwnd按线性规律缓慢增长。
* 为了防止cwnd增长过大，设置了一个慢开始门限ssthresh：
        当cwnd < ssthresh，使用慢开始算法
        当cwnd > ssthresh，停止慢开始使用拥塞避免
        当cwnd = ssthresh，既可以使用慢开始，也可以使用拥塞避免。
* **当网络出现超时时，发送方判断网络出现了拥塞，调整门限值ssthresh = cwnd / 2，同时设置cwnd = 1。**
* 快重传算法可以让发送方尽早知道个别报文的丢失，接收方立即发送确认，即使收到了失序的报文段也要立即发出对已收到的报文段的重复确认。当发送方一连收到3个重复确认，就立即进行重传。
* **当发送一连收到3个对同一个报文段的重复确认(3-ACK)时，执行快恢复算法，调整门限制ssthresh = cwnd / 2，cwnd = ssthresh，并执行拥塞避免算法。**
* 接收方根据自己的接收能力设定了接收方窗口rwnd并把这个窗口写入TCP首部的窗口字段，传送给发送方。**发送方的发送窗口一定不能超过对方给出的接收方窗口值rwnd。**
* **发送方的窗口上限值应当取接收方窗口rwnd和拥塞窗口cwnd这两个变量中较小的一个。**
#### TCP的运输连接管理

* 运输连接有三个阶段：连接建立、数据传送、连接释放。TCP连接的建立采用客户服务器方式，主动发起连接建立的应用进程叫客户client，而被动等待连接建立的应用进程叫做服务器server。
##### TCP的连接建立
![](https://github.com/dukezys/Computer-Network/blob/master/pics/TCPthree-way%20handshake.png)

* 一开始，B的TCP服务器进程创建传输控制块TCB，然后处于LISTEN状态。
* A的TCP客户进程也是先创建TCB，打算建立TCP连接时，向B发出连接请求报文段，这是**首部中SYN = 1**，同时选择一个**初始序列号seq = x。**TCP规定，SYN报文段不能携带数据，但要消耗一个序列号。这时TCP客户进入SYN-SENT状态。
* B收到连接请求后，在确认报文中**SYN和ACK都置为1，确认号ack = x + 1，初始序号 seq = y。**这个报文也不能携带数据，消耗一个序号。服务器进入SYN-RCVD状态。
* TCP客户进程收到B的确认后，还要向B给出确认。确认报文段的**ACK为1，确认号ack = y + 1，序号seq = x + 1。**ACK报文段可以携带数据，但**如果不携带数据则不消耗序号。**
* 为什么A最后还要发送一次确认？主要是为了防止已失效的连接请求报文段突然又传送到了B。
##### TCP的连接释放
![](https://github.com/dukezys/Computer-Network/blob/master/pics/TCPfour-way%20handshake.png)

* A的应用进程先向其TCP发出连接释放报文段，并停止再发送数据，主动关闭TCP连接。A把连接释放报文段首部的终止控制位**FIN置1，序号seq = u**，等于前面已传送的数据的最后一个字节的序号加1。A进入FIN-WAIT-1。
* B收到释放报文后立即发出确认，**确认号ack = u + 1，报文段序号为v**，等于B前面已传送过的数据的最后一个字节的序号加1。B进入CLOSE-WAIT状态。TCP服务器进程这时应通知高层应用进程，这时TCP连接处于半关闭状态，A到B的连接释放，但B到A还没有。
* A收到B的确认后，就进入FIN-WAIT-2的状态，等待B发出的连接释放报文段。
* 若B已经没有要向A发送的数据，其应用进程就通知TCP释放的连接。这时B发出的连接释放报文段必须使**FIN = 1，假定现在B的序号为w，B还必须重复上次已发送过的确认号ack = u + 1。**B进入LAST-ACK状态。
* A收到B的连接释放报文段后，必须对此发出确认，**ACK置1，确认号ack = w + 1，序号是seq = u + 1。**进入TIME-WAIT。
* 此时TCP还没有释放掉，必须经过时间等待计时器设置的2MSL后，A才进入CLOSED状态。MSL叫做最长报文段寿命，RFC793建议两分钟。
* TIME-WAIT等待的理由：
        1. 为了保证A发送的最后一个ACK报文段能够到达B。这个ACK报文可能会丢失，B接收不到，则会超时重传FIN+ACK报文段，A就能在2MSL时间内收到这个重传的报文段，接着A重传一次确认，重新启动2MSL计时器。
        2. 防止“已失效的连接请求报文段”出现。A在发送完最后一个ACK报文段后，再经过时间2MSL，就可以使本连接持续的时间内所产生的所有报文段都从网络消失。
* B只要收到了A发出的确认，就进入CLOSED状态。同样，B在撤销相应的传输控制块TCB后，就结束了这次的TCP连接。B结束连接的时间比A要早。


## 应用层
### 万维网WWW
#### 超文本传送协议HTTP

* 从层次的角度看，HTTP是面向事物(transaction-oriented)的应用层协议，是万维网上能够可靠地交换文件的重要基础。
* HTTP使用了面向连接的TCP作为运输层协议，保证了数据的可靠传输。HTTP协议本身是无连接的，虽然使用TCP连接，但通信双方在交换HTTP报文之前不需要先建立HTTP连接。
* HTTP协议是无状态的。同一个客户第二次访问同一个服务器上的页面时，服务器的响应与第一次被访问时的相同(假定现在服务器还没有把该页面更新)。
* 默认端口为80
* 建立TCP连接的三报文握手的前两部分完成后，万维网客户就把HTTP请求报文，作为建立TCP连接的三报文握手中的第三个报文的数据。**请求一个万维网文档所需的时间是该文档的传输时间加上两倍往返时间RTT。**
* HTTP/1.0缺点是每请求一个文档就要有两倍RTT开销。另一种开销是万维网客户和服务器每一次建立新的TCP连接都要分配缓存和变量。特别是万维网服务器往往要同时服务大量客户的请求，非持续连接会使万维网服务器的负担很重。
* HTTP/1.1使用了**持续连接(persistent connection)。** 就是万维网服务器在发送响应后仍然在一段时间内保持这条链接，使同一个客户和该服务器可以继续在这条连接上传送后续的HTTP请求报文和响应报文。不局限于传送同一个页面链接上的文档，在同一个服务器上即可。
* HTTP/1.1持续连接的两种工作方式：**非流水线方式（without pipelining)** 和**流水线方式(with pipelining)。**
* 非流水线方式：客户在收到前一个响应后才能发出下一个请求。在连接已建立后，客户每访问一次需要一个RTT。缺点是服务器发送完一个对象后，其TCP连接就处于空闲状态，浪费了服务器资源。
* 流水线方式的特点：客户在收到HTTP的响应报文之前就更能够接着发送新的请求报文，于是服务器就可以连续发回响应报文。客户访问所有的对象只需花费一个RTT时间。
##### HTTP的报文结构

* HTTP有两类报文：
    1) 请求报文———从客户向服务器发送请求报文
    2) 响应报文———从服务器到客户的回答
* HTTP请求报文和响应报文都是由三个部分组成的：
    1) 开始行，用于区分是请求报文还是响应报文。在请求报文中的开始行叫做**请求行(Request-Line)**，在响应报文中的开始行叫做**状态行(Status-Line)**。开始行的三个字段之间都以空格分隔开，最后的“CR”，“LF”分别代表“回车”和“换行”。
    2) 首部行，用来说明浏览器、服务器或报文主体的一些信息。首部可以有好几行，但也可以不使用。
    3) 实体主体，在请求报文中一般不用这个字段，响应报文中也可能没有这个字段。
* HTTP请求报文特点：请求报文第一行“请求行”只有三个内容：方法，请求资源的URL，HTTP的版本。方法是对所请求的对象进行的操作，实际上也就是一些命令。
* HTTP响应报文的特点：状态行包括三个内容，即HTTP的版本，状态吗，以及解释状态码的简单短语。状态码都是三位数字的，分5类：
     1) 1xx表示通知信息
     2) 2xx表示成功
     3) 3xx表示重定向
     4) 4xx表示客户的差错
     5) 5xx表示服务器的差错
#### 超文本传输安全协议HTTPS

* HTTPS是一种通过计算机网络进行安全通信的传输协议。HTTPS经由HTTP进行通信，但利用SSL/TLS来加密数据包。
* 对称加密：密钥只有一个，加密解密为同一个密码，且加解密速度快，典型的算法有DES、AES等。
* 非对称加密：密钥成对出现，加密解密使用不同密钥，相对对称加密速度较慢，典型的非对称加密算法有RSA、DSA等。
