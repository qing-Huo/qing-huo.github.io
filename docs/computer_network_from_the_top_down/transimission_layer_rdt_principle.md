## 可靠数据传输(RDT)的原理

- rdt在应用层,传输层和数据链路层都很重要
- 时网络TOP10问题之一
- 信道的不可靠特点决定了可靠数据传输协议(rdt)的复杂性


#### 可靠数据传输:问题描述(续)

- 我们将早如下推导:
- 渐增式地开发可靠数据传输协议(rdt)的发送方和接收方
- 只考虑单项数据传输
	* 但控制信息是双向流动的
- 双向的数据传输问题实际上是2个单向数据传输问题的综合
- 使用有限状态机(FSM)来描述发送方和接收方


#### Rdt1.0: 在可靠信道上的可靠数据传输

- 下层的信道是完全可靠的
	* 没有比特出错
	* 没有分组丢失
- 发送方和接收方的FSM
	* 发送方将数据发送到下层信道
	* 接收方从下层信道接收数据

#### Rdt2.0:具有比特差错的信道

- 下层信道可能会出错:将分组中的比特翻转
	* 用校验和来检测比特差错
- 问题: 怎样从差错中恢复:
	* 确认(ACK): 接收方显示地告诉发送方分组已被正确接收
	* 否定确认(NAK): 接收方显式地告诉发送方分组发生了差错
		- 发送方收到NAK后,发送方重传分组
- Rdt2.0中的新机制: 采用差错控制编码进行差错检测
	* 发送方差错控制编码,缓存
	* 接收方使用编码检错
	* 接收方的反馈: 控制报文(ACK,NAK): 接收方 -> 发送方
	* 发送方收到反馈后,相应的动作

#### Rdt2.0的致命缺陷! 

- 如果ACK/NAK出错?
- 发送方不知道接收方发生了什么事情!
- 发送方如何做?
	* 重传? 可能重复
	* 不重传? 可能死锁(或出错)
- 此时需要引入新的即使
	* 序号

- 处理重复:
	* 发送方在每个分组中加入序号
	* 如果ACK/NAK出错,发送方```重传```当前分组
	* 接收方丢弃(不发给上层)重复分组

- 停等协议
	* 发送方发送一个分组,然后等待接收方的应答


#### Rdt2.1: 讨论

```
发送方:
	在分组中加入序列号
	两个序列号(0,1)就足够
		一次只发送一个未经确认的分组
	必须检测ACK/NAK是否出错(需要EDC)
	状态数变成了两倍
		必须记住当前分组的序列号为0还是1

接收方:
	必须检测接收到的分组是否是重复的
		状态会指示希望接收到的分组的序号为0还是1

note: 接收方并不知道发送方是否正确收到了其ACK/NAK
	没有安排确认的确认

接收方不知道它最后发送的ACK/NAK是否被正确地收到
发送方不对收到的ack/nak给确认	(没有所谓的确认的确认)
接收方发送ack,如果后面接收方收到的是:
	老分组(序号为)P0? 则ack错误
	下一个分组? P1,则ack正确
```

#### Rdt2.2: 无NAK的协议

- 功能同Rdt2.1,但只是用ACK(ack要编号)
- 接收方对```最后```正确接收的分组发ACK,以代替NAK
	* 接收方必须显示地包含被正确接收分组的```序号```
- 当收到重复的ACK(如: 再次收到ack0)时,发送方与收到NAK采取相同的动作: ```重传当前分组```
- 为最后的一次发送多个数据单位做一个准备
	* 一次能够发送多个
	* 每一个的应答都有: ACK,NCK,麻烦
	* 使用对前一个数据单位的ACK,代替本数据单位的NAK
	* 确认信息减少一般,协议处理简单

#### Rdt3.0:具有比特差错和分组丢失的信道

- 新的假设: 下层信道可能会丢失分组(数据或ACK)
	* 会死锁
	* 机制还不够处理的状况如下
		- 检验和
		- 序列号
		- ACK
		- 重传

- 方法: 发送方等待ACK一段```合理时间```
	* 发送端超时重传: 如果到时没有收到ACK -> 重传
	* 问题: 如果分组(或ACK)只是被延时了:
		- 重传将会导致数据重复,但利用```序列号```已经可以处理这个问题
		- 接收方必须指明被正确接收的序列号
	* 需要一个倒计数定时器

#### 流水线协议

- 流水线: 允许发送方在```未得到对方确认```的情况下一次发送```多个```分组
	* 必须增加序号的范围: 用多个bit表示分组的序号
	* 在发送方/接收方要有缓冲区
		- 发送方缓冲: 未得到确认,可能需要重传
		- 接收方缓存:上层用户取用数据的速率!=接收到的数据速率
			* 接收到的数据可能乱序,排序交付(可靠)
- 两种通用的流水线协议:
	* 回退N步(GBN)和选择重传(SR)

#### 通用:滑动窗口(slide window)协议

```
发送缓冲区
	形式: 内存中的一个区域,落入缓冲区的分区可以发送
	功能: 用于存放已发送,但是没有得到确认的分组
	必要性: 需要重发时可用
发送缓冲区的大小: 
	一次最多可以发送多少个未经确认的分组
	停止等待协议=1
	流水线协议>1,合理的值,不能很大,链路利用率不能找过100%
发送缓冲区中的分组
	未发送的: 落入发送缓冲区的分组,可以连续发送出去
	已经发送出去的,等待对方确认的分组: 发送缓冲区的分组只有得到确认才能删除
```


```
发送窗口: 发送缓冲区内容的一个范围
	哪些已经发送但是未经确认分组的序号构成的空间

发送窗口的最大值 <= 发送缓冲区的值

一开始: 没有发送任何一个分组
	后沿 = 前沿
	之间的发送窗口的尺寸 = 0

每发送一个窗口,前沿前移一个单位
```

- 发送窗口的移动 -> 后沿移动
	* 条件: 收到老分组(后沿)的确认
	* 结果: 发送缓冲区罩住新的分组,来了分组可以发送
	* 移动的极限: 不能找过前沿

- 滑动窗口协议 - 接收窗口

```
接收窗口(receiving window) = 接收缓冲区
	接收窗口用于控制哪些分组可以接收
		只有收到的分组序号落入接收窗口内才允许接收
		若序号在接收窗口之外,则丢弃
	接收窗口尺寸Wr=1,则只能顺序接收
	接收窗口尺寸Wr>1,则可以乱序接收
		但提交给上层的分组,要按序
```

- 正常情况下的2个窗口互动

```
发送窗口
	有新的分组落入发送缓冲区, 发送 -> 前沿滑动
	来了老大低序号分组的确认 -> 后沿向前滑动 -> 新的分组可以落入发送缓冲区的范围
接收窗口
	收到分组,落入到接收窗口范围内,接收
	是低序号,发送确认给对方

发送端上面来了分组 -> 发送滑动窗口 -> 接收窗口滑动-> 发确认
```
- 异常情况下GBN的2个窗口互动

```
发送窗口
	新分组落入发送缓冲区, 发送 -> 前沿滑动
	超时重发机制让发送端将发送窗口中的所有分组发送出去
	来了老分组的重复确认 -> 后沿不向前滑动 -> 新的分组无法落入发送缓冲区的范围
		(此时如果发送缓冲区由新的分区可以发送)
接收窗口
	收到乱序分组,没有落入到接收窗口范围内,抛弃
	(重复)发送老分组的确认,累计确认
```

- 异常情况下SR的2个窗口互动

```
发送窗口
	新分组落入发送缓冲区 发送 -> 前沿滑动
	超时重发机制让发送端将超时的分组重新发送出去
	来了乱序分组的确认 -> 后沿不向前滑动 -> 新的分组无法落入发送缓冲区的范围
		此时如果发送缓冲区有新的分组可以发送
接收窗口
	收到乱序分组,落入到接收窗口范围内,接收
	发送该分组的确认,单独确认
```

- GBN协议和SR协议的异同

```
相同之处
	发送窗口 > 1
	一次能够可发送多个未经确认的分组

不同之处
	GBN: 接收窗口尺寸 = 1
		接收端: 只能顺序接收
		发送端: 从表现来看,一旦一个分组没有发送成功,如: 0,1,2,3,4
			假如1未成功,234都发送出去了,要返回1再发送,GB1
	SR: 接收窗口尺寸 > 1
		接收端: 可以乱序接收
		发送端: 发送0,1,2,3,4,一旦1未成功,2,3,4已发送
			无需重发,选择性发送1	
```

#### 流水线协议总结

- GO-back-N:
	* 发送端最多再流水线中有N个未确认的分组
	* 接收端只是发送累计型确认cumulative ACK
		* 接收端如果发现gap,不确认新到来的分组
	* 发送端拥有对最老的未确认分组的定时器
		- 只需设置```一个定时器```
		- 当定时器到时时,```重传所有未确认分组```
- Selective Repeat:
	* 发送端最多在流水线中有N个未确认的分组
	* 接收方对每个到来的分组单独确认```individual ACK(非累计确认)```
	* 发送方为每个```未确认的分组保持一个定时器```
		- 当超时定时器到时,只是重发到时的未确认分组


###### 对比GBN和SR

```
			GBN				SR
优点		简单,所需资源少		出错时,重传一个代价小
		接收方一个缓存单元	

缺点	   一旦出错,回退N步代价大	 复杂,所需资源多(接收方多个缓存资源)

适用范围
	出错率低: 
		适合GBN,出错非常罕见,不需要复杂的SR
	链路容量大(延迟大,带宽大):
		适合SR而不是GBN,一点出错代价太大
```























