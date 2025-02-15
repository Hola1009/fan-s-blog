为什么要有?
为了解决不同设备之间的通信问题

有五层 应用层, 传输层, 网络层, 网络接口层(数据链路层)
### 应用层
应用层只需要关注为用户提供功能, 比如http; 不需要关注数据是如何传输的
应用层工作在操作系统中的用户态, 传输层及一下则工作在内核态
### 传输层
应用层的数据包会传给传输层, 传输层是为应用层提供网络支持的, 应用层的数据到了传输层会被加上报头 形成新的报文 再传给下一层
接下来说一说传输层的协议: 传输层的协议有 TCP 和 UDP 特性分别为 可靠通信 和 不可靠通信
TCP 相比 UDP 多了很多特性 如 流量控制, 超时重传, 拥塞控制等, 都是其可靠传输的保障
因为 UDP 只负责发数据就完了 所以虽然其不可靠传输, 但是效率高

应用传输的数据可能非常大, 直接传输不好控制, 因此就要将数据包分块, 一个包丢失了就只重传丢失的即可, 再tcp协议中我们把每个分块 称为一个TCP段

### 网络层
传输层的设计理念是简单, 高效, 但是网络中的节点是错综复杂的, 一个设备要传数据给另一个设备, 需要各种各样的路径和节点进行选择, 这与传输层的设计原则冲突, 所以实际的传输功能就要就给下一层 网络层
网络层用的协议是ip协议, 根据ip协议, 它会将传输层传来的报文加上ip报头组成新的报文, 
网络层怎么区分设备编号呢?
用ip地址 + 掩码 进行设备区分
用ip地址和掩码进行运算可以得到 网络号 和 主机号
网络号加主机号就区分出了一台设备, 传相互的时候就根据网络号和主机号来寻址
寻址的时候先根据网络号找到子网, 再找对应的主机, 
除了寻址, ip协议另个重要功能就是路由, 两台设备间的网络链接不是一根网线那样简单, 而是靠各种网络节点转来转去这样链接的, 当一个网络节拿到数据报后, 就会根据路由算法决定下一步路该怎么走
**IP 协议的寻址作用是告诉我们去往下一个目的地该朝哪个方向走，路由则是根据「下一个目的地」选择路径。寻址更像在导航，路由更像在操作方向盘**。

### 网络接口层
生成了 IP 头部之后，接下来要交给**网络接口层**（_Link Layer_）在 IP 头部的前面加上 MAC 头部，并封装成`数据帧（Data frame）`发送到网络上。在以太网上传输
在以太网上传输也有一套传输协议, 所以还需要堆网络层传来的数据进行一次打包, 加上MAC报头组成新的数据帧
