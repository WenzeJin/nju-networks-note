# 链路层

## 功能与服务

提供的服务：成帧、链路介入、可靠交付、差错检测和纠错。后两个是可选的。

## 差错检测

简单的方法有奇偶检测，一般用CRC（循环冗余检测）

为什么链路层可以用CRC，而TCP采用检验和呢？因为链路层一般与硬件相关联，我们可以在硬件中实现高效的CRC计算。

CRC 要求我们先确定一个 r+1位的模式，第一位必须是 1 不然位数没有意义。其次我们按照以下公式计算 R（r位的）并附在原数据后，注意在除法中所有的加减运算均为模-2运算，即均为 NOR 运算，下面是公式和一个简单的例子：

$$
R = \text{remainder} \frac{D \cdot 2^R}{G}
$$

<figure><img src=".gitbook/assets/截屏2024-06-13 19.58.44.png" alt="" width="283"><figcaption><p>上式的计算过程</p></figcaption></figure>

## 多路访问协议 MAC

通过这些协议来规范物理设备在共享的广播信道上的传输行为

主要分为三类： 信道划分协议 、随机接入协议和轮流协议

### **信道划分协议**

TDM、FDM

### **随机接入协议**

#### **ALOHA**

ALOHA在夏威夷语中具有打招呼的含义。

其处理碰撞的方式非常简单，先传输，检测到碰撞即准备重传，每个时间间隔以 p 的概率重传。

#### **时隙ALOHA**

将时间划分为长度为 L/R 的时隙，L 为帧长度，我们令所有帧长度相同 若假设所有端系统均在重传状态，有N个，概率为p，则一个成功时隙的概率为 $$Np(1-p)^{N-1}$$ 。当N很大时， $$p = \frac{1}{N}$$令效率最高。

**载波帧听 CSMA**

端到端信道传播时延 (channel propagatioo delay) (信号 一个节点传播到另一个节点所花费的时间)在决定其性能方面起着关键 的作用 。 该传播时延越长，载波侦听节点不能侦听到网络中另 一个节点已经开始传输的机会就越大。

对于全双工设备组成的局域网，可以采用 CSMA/CD (CSMA with Collision Detection)：

简要阐述：若侦听到信道空闲，（等待 IFS）传输帧，若发送过程中侦听到碰撞，停止并随即回退。

几个细节：

1. 随机回退的规则：在该帧经历了 一连串的 n 次碰撞后，节点随机地从 {0, l , 2 , ... , 2^n - 1} 中选择 一个 K值。 因此， 一个帧经历的碰撞越多， k选择的间隔越大。
2. 报文长度范围：报文传输时间为 L/R 它必须要大于检测到可能碰撞的时间，假设最远的一个端系统距离发送方距离为 D，传播速度为 V，假设在该帧到达其的前一刻（非常短）时间内，该端系统发送了一个帧，则立即发生碰撞，但是当发送方检测到这一碰撞，已经是开始发送后的 2D/V 时间了，所以我们有：$$\frac{L}{R} \ge \frac{2D}{V}$$ 也就是 $$L \ge \frac{2D\cdot R}{V}$$ 这就得到了最小帧长，反过来我们也可以求最大范围。

### **轮流协议**

#### 令牌环

令牌在环上传递，需要发送帧的端系统需要先获取令牌，然后附在要发送的帧上。经过一圈后取下令牌并释放。一种集中式的协议（“集中”在这个令牌上），需要防止单点失效（e.g. 丢失令牌）。

## MAC 协议评价与比较

<figure><img src=".gitbook/assets/截屏2024-06-13 21.36.18.png" alt="" width="563"><figcaption></figcaption></figure>

定义：&#x20;

1. **"1"** 归一化的传输时间&#x20;
2. $$a$$: 与之相对应的传播时间&#x20;
3. $$N$$: 参与的端系统数量

那么对于：

### **点对点传输**

$$
U = \frac{1}{1 + a}
$$

### **令牌环**

$$T_1$$ 平均传输时间: 传输后直到释放令牌的时间（转完一圈）

不难得到 $$U = \frac{T_1}{T_1 + T_2}$$ 。但是我们需要分两种情况去考虑 $$T_2$$&#x20;

首先是帧传输时间大于转一圈时间，此时令牌会在传输完后立即被释放，直到到达下一跳，也就是 a < 1 的情况 $$U = \frac{1}{1 + \frac{a}{N}}$$&#x20;

其次就是帧传输时间小于转一圈时间，此时首先要等一圈传完，这个等待时间是 a - 1, 之后还要加上到下一跳的时间，所以 a > 1 时，$$U = \frac{1}{a + \frac{a}{N}}$$

### **ALOHA** SLOTTED

首先，ALOHA的时隙长度应该为 1 + 2a，因为需要预留a用于检测碰撞，同时用 a 用于确保最后一个比特的传输，那么结合我们上述的概率，假设概率为 A，令 $$U_s = \frac{1}{1 + 2a}$$那么 $$U = A \cdot U_s \approx A$$ 则根据我们上述对ALOHA的分析，U的最大值应该为 \$$\frac{1}{e}\$$.

### **ALOHA**

修改 ALOHA SLOTTED中关于概率的描述，此时概率为$$A = Np(1-p)^{2N - 1}$$最大值为 $$U_s = \frac{1}{2e}$$

### **CSMA/CD**&#x20;

此协议的性能分析细节不需要掌握，只需要背一下如下公式：

$$U = \frac{1}{1 + 4.44a}$$

## 局域网 LAN

一种是上面提到的令牌环结构

一种是 Ethernet，以太网又可以分为：总线结构和交换结构（星型）

总线结构的以太网使用 CSMA/CD

交换结构的以太网则不需要使用 CSMA/CD，直接通过记忆和生成树算法（防止环路）进行路由

以太网是无连接、不可靠的

<figure><img src=".gitbook/assets/截屏2024-06-13 21.18.05.png" alt="" width="563"><figcaption></figcaption></figure>

引入 MAC地址的概念，注意MAC地址的MAC是 Medium Access Control 的缩写，和MAC协议的含义并不完全一样 12位十六进制数

### **Discover self in LAN: ARP & DHCP**

Link layer discovery protocols

* ARP : Address Resolution Protocol
* DHCP : Dynamic Host Configuration Protocol
* Confined to a single local-area network (LAN)
* Rely on broadcast capability

#### ARP

ARP request and reply: 获取 MAC 地址 如果目标不在子网内（可以通过子网掩码得知这一点），那么就给默认网关发送 ARP 请求获取他的 MAC 地址

#### DHCP：

获取 IP、子网掩码、DNS服务器IP、默认网关IP

#### 流程：



<figure><img src=".gitbook/assets/截屏2024-06-15 15.05.20.png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/截屏2024-06-15 15.05.53.png" alt=""><figcaption></figcaption></figure>

## 扩大局域网

以往使用网桥链接不同的 LAN 以扩展他们，现在可以使用交换机。

网桥的功能：

1. 存储转发：转发帧，可以是广播，也可以根据此前帧的信息学习与接口的对应关系，转发到对应的接口
2. 透明的：网桥在局域网中是透明的，其他设备感知不到它的存在
3. 即插即用：网桥不需要预先配置，是自学习的

实现上述功能的机制：

1. Frame Broadcast
2. Loop Resolution
3. Address Learning

### **Frame Broadcast**

<figure><img src=".gitbook/assets/截屏2024-06-15 15.13.51.png" alt="" width="563"><figcaption></figcaption></figure>

问题：广播风暴

解决：在拓补中消除环路

### **Loop Resolution**

生成树算法显然可以解决这个问题，但是我们不使用经典的 MST 算法：

1. 我们不需要最小生成树
2. 我们需要分布式算法

两个要点：

1. 选取源点
   1. 我们研究的是其他点到源点的最段路径
   2. 选择序号（MAC）最小的作为源点
2. 计算最短路径
   1. 所有点需要获取到源点的最短路径并选择它
   2. 最短路径不会产生环路

在此框架下细化：

Messages (Y, d, X)

* From node X
* Proposing Y as the root
* And advertising a distance d to Y
* Switches elect the node with smallest identifier (MAC address) as root Each node determines if a link is on its shortest path to the root; excludes it from the tree if not

继续细化，形成算法：

<figure><img src=".gitbook/assets/截屏2024-06-15 15.24.25.png" alt="" width="563"><figcaption></figcaption></figure>

一个例子：

<figure><img src=".gitbook/assets/截屏2024-06-15 15.27.39.png" alt="" width="563"><figcaption></figcaption></figure>

为了确保鲁棒性，必须要应对单点失效：

根结点周期性地发送消息，其他节点如果有一段时间没有收到根结点的消息了，就将自己作为根结点，并重新引发上述计算

现在，考虑如何在生成树上转发：

* _**(Ignore all ports not on spanning tree!)**_
* Originating bridge sends packet out all ports。
* When a packet arrives on one incoming port, send it out all ports other than the incoming port。

会产生很多广播的浪费，所以我们引入自学习机制。

### **Address Learning**

<figure><img src=".gitbook/assets/截屏2024-06-15 15.33.17.png" alt="" width="563"><figcaption></figcaption></figure>

具体的方式将会在Lab2中实现，这里就不赘述了。

现在，继续讨论与总结扩展 LAN 的设备：

| 设备               | 描述                                                           |
| ---------------- | ------------------------------------------------------------ |
| Hubs             | physical repeaters                                           |
| Bridges          | connecting LANs (forwarding + address learning)              |
| Layer 2 switches | connecting Hosts or LANs (bridge functions + collision free) |
| Layer 3 switches | involving router functions                                   |

**重点看以下两种设备**

**Hub:** Repeater，在物理上是星型拓补，但是在逻辑上还是总线结构，因为所有人共享Hub，如果有二者想要同时传输数据，会产生碰撞

**L2 Switches:** 在物理上是星型拓补，可以直接链接主机或局域网，可以同时在不同的端系统对之间传输数据，不会产生碰撞，具有记忆功能，不必每次都广播



<figure><img src=".gitbook/assets/截屏2024-06-15 15.39.59.png" alt="" width="563"><figcaption></figcaption></figure>

二者传输效率的比较，可以从上图看出，Hub仅支持将一个接口上到来的帧转发到其他所有接口，所以一次只能为一个帧服务，但是switch可以创建多个一对一连接，可以同时为两组或更多通信服务。

当然，这些设备依然需要生成树机制。

## 无线网络

组成无线网络的成员： 无线主机、基站、无线链路

两种模式： 基础设施模式 自组织模式

无线网络的问题： 误码率高：BER 随着信噪比降低而升高，同样因为传输速率的增加而增大 多径传播 干扰

无线网络的特点： 广播媒介 半双工 无序到达 其他问题

### **802.11**

#### **结构：**

AP：接入点，它是一个无线局域网络中的设备，用于连接无线客户端设备（如笔记本电脑、智能手机、平板电脑等）到有线局域网络（LAN）或广域网络（WAN），总称为分布式系统 Distribution System。AP通常连接到有线网络中，并充当无线网络的中心节点。

**BSS：** 基本服务集。BSS是无线局域网络（WLAN）中的一个基本组成单元，它包括一个或多个无线设备（通常是客户端设备）和一个AP（Access Point）。

**ESS：** ESS是由多个基本服务集（BSS）组成的更大的逻辑集合，它通过一个公共的分布式系统（Distributed System）相互连接。一个ESS在逻辑上是一个局域网 LAN，利用网关与Internet 连接

DS：用于连接各个BSS和LAN来创造一个 ESS



<figure><img src=".gitbook/assets/截屏2024-06-15 16.12.32.png" alt="" width="560"><figcaption><p>802.11 示意图</p></figcaption></figure>

#### 频段

在 802.11b/g 协议中使用 2.4GHz-2.485GHz，分为 11个信道。AP admin 为 AP 选择信道。可能产生干扰 。速率：11Mbps(b) - 54Mbps。

每个主机必须与一个AP连接： 帧听来自 AP 的 beacon frames 包含 SSID 和 MAC 选择一个喜欢的 AP 连接，可能需要认证，并运用 DHCP 获得各种信息。AP 也就是我们家中熟知的“无线路由器”的一部分功能。

### Multiple Access in 802.11

现在我们考虑 802.11 中的 媒体接入（MAC）

**在以太网中使用的 CSMA/CD 不可用：**

1. 无线局域网主要使用的是半双工的技术
2. **可能有以下三个问题，使得潜在的干扰无法被检测到：**

<figure><img src=".gitbook/assets/截屏2024-06-15 16.20.45.png" alt="" width="563"><figcaption><p>在WLAN中应用CSMA/CD可能的三个问题</p></figcaption></figure>

#### **应用 RTS/CTS 解决 Hidden Terminal 问题**

1. Source: RTS request to send 发送“请求发送”帧
2. Destination: CTS clear to send 发送“准许发送”帧
3. Source: transmits data
4. Destination: ACK

RTS CTS 实际上是对各自传输范围内主机的警告。

<figure><img src=".gitbook/assets/截屏2024-06-15 16.29.23.png" alt="" width="563"><figcaption><p>RTS/CTS 机制示意图</p></figcaption></figure>

既是确认，又是警告：在听到 B 的 CTS 之后，C应当保持静默，以免干扰。

#### **接下来解决 MAC 的其他问题**

技术：DWFMAC 分布式基础无线媒体访问控制，主要机制如下，

PCF Point Coordination Function 点协调功能，集中式控制，用于指挥发送实时数据，优先级低于控制帧。

DCF Distributed Coordination Function 分布式协调功能，分布式控制，用于传输异步数据，优先级最低。

**综合协调：** DWFMAC 提供了两种不同的协调机制（PCF和DCF），可以根据不同的数据类型和实时性要求选择合适的传输方式。

PCF负责实时数据的高优先级传输，集中式的调度机制可以有效避免冲突。

DCF适用于传输低优先级的数据，分布式的控制方式虽然可能带来更多的冲突和延迟，但能够降低对中央协调者的依赖。

**优先级差异：** DWFMAC通过区分 PCF 和 DCF 的优先级，实现了对不同类型数据的优先级调度。实时数据使用PCF，异步数据使用DCF，从而优化了带宽的利用率和网络的整体性能。当然，用于控制的控制帧优先级比二者都高。

**如何实现上述优先级？采用帧间不同的间隔：**

SIFS Short Inter Frame Space：用于传输即时的控制信息，最高优先级 PIFS Pcf IFS：用于传输 PCF 消息 DIFS Dcf IFS：用于传输 DCF 消息

<figure><img src=".gitbook/assets/截屏2024-06-15 16.40.56.png" alt="" width="563"><figcaption><p>三种优先级的直观图示</p></figcaption></figure>

<figure><img src=".gitbook/assets/截屏2024-06-15 16.42.07.png" alt="" width="321"><figcaption><p>三种优先级的应用逻辑</p></figcaption></figure>

**引入超级帧：**

点协调器不断发布轮询，会封锁所有异步通信量。为了避免这种情况，在超帧时间的前一部分，由点协调器轮询，在超帧时间的后一部分，允许异步通信量争用接入。

#### **DCF 采用什么机制？CSMA/CA**

CA：Collision Avoidance

在 DIFS 后，感知媒体的状态，如果空闲就传输，否则随机退避

**为什么不采用 CSMA/CD？**

1. 半双工；
2. 即使全双工，也很难感知到碰撞，因为反射回来已经很弱了；
3. 无法区分噪声和自身传输的效应；
4. 无法感知到所有碰撞：考虑我们上面说的一些问题；
5. 成功的传输会有 ACK 回应，无需监听。

<figure><img src=".gitbook/assets/截屏2024-06-15 16.55.29.png" alt=""><figcaption><p>CSMA/CA 的时序图</p></figcaption></figure>

可以看到，时序图中，发送方等待了一个 DIFS 并发送 data，接收方此时发送的是 ACK，是一个控制帧，所以等待的时隙长度为 SIFS。
