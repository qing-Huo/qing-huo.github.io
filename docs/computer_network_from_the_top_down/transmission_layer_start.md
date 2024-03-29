## 传输层概述

目标:
- 理解传输层的工作原理
	* 多路复用/解复用
	* 可靠数据传输
	* 流量控制
	* 拥塞控制
- 学习Internet的传输层协议
	* UDP:无连接传输
	* TCP:面向连接的可靠传输
	* TCP的拥塞控制

#### 传输服务和协议

- 为运行在不同主机上的应用进程提供```逻辑通信```

```
传输协议运行在端系统
	发送方:将应用层的报文分成报文段,然后传递给网络层
	接收方:将报文段重组成报文,然后传递给应用层

有多个传输层协议可供应选择
	Internet: TCP和UDP
```

#### 传输层 VS 网络层

- 网络层服务:主机之间的逻辑通信
- 传输层服务:进程间的逻辑通信
	* 依赖于网络层的服务(延时,带宽)
	* 并对网络层的服务进行增强(数据丢失,顺序混乱,加密)
- 有些服务时可以加强的:不可靠 -> 可靠
- 有些服务不可以被加强: 带宽,延迟

#### Internet传输层协议

- 可靠的,保序的传输: TCP
	* 多路复用/解复用
	* 拥塞控制
	* 流量控制
	* 建立连接
- 不可靠,不保序的传输: UDP
	* 多路复用/解复用
	* 没有为尽力而为的IP服务添加更多的其他额外服务
- 都不提供的服务
	* 延时保证
	* 带宽保证



























