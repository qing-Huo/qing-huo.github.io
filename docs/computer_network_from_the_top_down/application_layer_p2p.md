## P2P (Peer to Peer)
- P2P分为非结构化与结构化
- 非结构化：
  * 集中化目录
  * 完全分布式
  * 混合体
- 结构化:

### 纯P2P架构
- 没有(或极少)一直运行的服务器
- 任意端系统都可以直接通信
- 利用Peer的服务能力
- Peer节点间歇上网，每次IP地址都可能变化
- 例子：
  * 文件发布(BitTorrent)
  * 流媒体(KanKan)
  * VoIP(Skype)

### P2P文件共享
- 例子：
  * Alice在PC上运行P2P客户端
  * 间歇性的连接到Internet，每次从ISP得到新的IP地址
  * 请求 “Music.mp3”
  * 客户端显示其他有“Music.mp3”拷贝的对等放
  * Alice选择其中一个对等方,如Bob
  * 文件从Bob的PC传送到Alice的PC上(HTTP)
  * 当Alice下载时，其他用户也可以从Alice处下载
  * Alice的对等方既是一个Web客户端，也是一个瞬时Web服务器
- 所有的对等方都是服务器 = 可扩展性好

### ```两大问题：```
  * 如何定位所需资源
  * 如何处理对等方的加入与离开

## 非结构化P2P

### P2P:集中式目录
1. 当对等方连接时，它告知中心服务器：
  * IP地址
  * 内容
2. Alice查询“Music.mp3”
3. Alice从Bob处请求文件

- 集中式目录存在的问题
  * 单点故障
  * 性能瓶颈
  * 侵犯版权
- 文件传输是分散的，而定位内容则是高度集中的

## 完全分布式
### 查询洪泛：Gnutella
- 全分布式
  * 没有中心服务器
- 开放文件共享协议
- 许多GGnutella客户端实现了Gnutella协议
  * 类似HTTP有许多的浏览器

- ```覆盖网络：图```
  * 如果X和Y之间有一个TCP连接，则二者之间存在一条边
  * 所有活动的对等方和边就是覆盖网络
  * 边并不是物理链路
  * 给定一个对等方，通常所连接的节点少于10个

#### Gnutella:协议
- 在已有的TCP连接上发送查询报文
- 对等方转发查询报文
- 以反方向返回查询命中报文

##### Gnutella：对等方加入
1. 对等方X必须首先发现某些已经覆盖网络中的其他对等方：
  * 使用可用对等方列表
  * 自己维持一张对等方列表(经常开机的对等方的IP)
  * 联系维持列表的Gnutella站点
2. X接着试图与该列表上的对等方建立TCP连接，直到与某个对等方Y建立连接
3. X向Y发送一个Ping报文，Y转发该Ping报文
4. 所有收到Ping报文的对等方以Pong报文响应
  * IP地址，共享文件的数量及总字节数
5. X收到许多Pong报文，然后他能建立其他TCP连接

### 混合体 利用不匀称性：KaZaA
- 每个对等方要么是一个组长，要么隶属于一个组长
  * 对等方与其组长之间有TCP连接
  * 组长对之间有TCP连接
- 组长跟踪其所有的孩子的内容
- 组长与其他组长联系
  * 转发查询到其他组长
  * 获得其他组长的数据拷贝

- KaZaA：查询
  * 每个文件有一个散列标识码和一个描述符
  * 客户端向其组长发送关键字查询
  * 组长用匹配进行响应
    - ```对每个匹配：元数据，散列标识码和IP地址```
  * 如果组长将查询转发给其他组长，其他组长也以匹配进行响应
  * 客户端选择要下载的文件
    - ```向拥有文件的对等方发送一个带散列标识码的HTTP请求```

#### P2P文件共享：BitTorrent
- Peer加入torrent：
  * 一开始没有块，但是会通过其他节点处累计文件块
  * 向跟踪服务器注册，获得peer节点列表，和部分peer节点构成邻居关系("连接")
- 当peer下载时，该peer可以同时向其他节点提供上载服务
- Peer可能会变换用于交换块的peer节点
- ```扰动churn：```peer节点可能会上线或者下线
- 一旦一个peer拥有整个文件，它会(自私的)离开或者保留(利他主义)在torrent中
