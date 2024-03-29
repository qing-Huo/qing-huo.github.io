## 多路复用和解复用

- 在发送方主机多路复用

```
从多个套接字来接收来自多个进程的报文,
根据套接字对应的IP地址和端口号等信息对报文段用
 头部加以封装
(该头部信息用于以后的解复用)
```
- 在接收方主机多路解复用

```
根据报文段的头部信息中的IP地址和端口号将
接收到的报文段发给正确的套接字(和对应的应用进程)
```
- 一个进程对应一个socket


#### 无连接的多路复用
- 当主机即受到UDP段时:
	* 检查UDP段中的目标端口号
	* 将UDP段交给具备那个端口号的套接字
	* 具备相同```目标IP地址```和```目标端口号,即源IP地址```或/
	* 且```源端口号```不同的IP数据报,将会被传到相同的目标UDP套接字上


## 无连接传输:UDP

- UDP(User Datagram Protocol) [RFC 768]
- "no frills," "bare bones" Internet传输协议
- '尽力而为'的服务,报文段可能
	* 丢失
	* 发送到应用程序的报文段乱序
- 无连接
	* UDP发送端和接收端之间没有握手
	* 每个UDP报文段都被独立的处理
- UDP应用领域
	* 流媒体(丢失不敏感,速率敏感,应用可控制传输速率)
	* DNS
	* SNMP
- 在UDP上实现可靠传输
	* 在应用层增加可靠性
	* 应用特定的差错恢复

#### UDP校验和

- 目标:检测在被传输报文段中的差错(比如比特反转)

```
发送方:
	将报文段的内容是为16比特的整数
	校验和: 报文段的加法和(1的补运算)
	发送方将校验和放在UDP的校验和字段

接收方:
	计算接收到的报文段的校验和
	检查计算出的校验和与校验和字段的内容是否相等
		不相等: 检测到差错
		相等: 没有检测到差错,但也许还是有差错
			残存错误
```

- Internet校验和的例子	
	* 注意: 当数字相加时,在最高位的进位要回卷,在加到结果上

```
两个16比特的整数相加
		1 1 1 0 0 1 1 0 0 1 1 0 0 1 1 0
		1 1 0 1 0 1 0 1 0 1 0 1 0 1 0 1

回卷   1 1 0 1 1 1 0 1 1 1 0 1 1 1 0 1 1

和	  1 0 1 1 1 0 1 1 1 0 1 1 1 1 0 0

校验和   0 1 0 0 0 1 0 0 0 1 0 0 0 0 1 1


目标端: 校验范围+校验和=1111111111111111通过校验
	否则没有通过校验
note:求和时,必须将进位回卷到结果上
```















