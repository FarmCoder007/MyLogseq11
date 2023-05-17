- ##  1. 网络分层
  collapsed:: true
	- 1. TCP/IP四层网络模型
	  应用层/传输层/网络层/链路层
	- 2. OSI七层网络模型
	  应用层/表示层/会话层/传输层/网络层/数据链路层/物理层
	- collapsed:: true
	  3. 传输层协议: TCP UDP
		- TCP协议 Tansport Control Protocol
			- 基于连接
			- 保证顺序 可靠 段编号和确认号
			- 一对一通信
			- 面向字节流
			- 首部开销大 20-60字节
		- UDP User Data Protocol
			- 无连接
			- 不保证顺序 不可靠
			- 可以一对多通信
			- 面向报文
			- 首部开销小 8字节
- ##  2.TCP三次握手
  collapsed:: true
	- 第一次握手: SYN(Seq=x Ack=0)
	  第二次握手: SYN+ACK(Seq=y Ack=x+1)
	  第三次握手: ACK(Seq=x+1 Ack=y+1)
	- ![image.png](../assets/image_1684305112067_0.png)
		- collapsed:: true
		  1. Wireshark 抓包查看握手过程
			- ![image.png](../assets/image_1684305152816_0.png)
		- 2、TCP首部 标记位 SYN ACK PSH FIN URG RST
		  collapsed:: true
			- SYN Synchronize 用于建立连接
			- ACK Acknowledge 用于确认
			- PSH Push 数据续写
			- FIN Finish 用于断开连接
			- URG Urgent 紧急标记
			- RST Reset 重置
			- ![image.png](../assets/image_1684305187686_0.png)
		- 3、Seq和Ack的值
		  collapsed:: true
			- Seq Sequence Number 段编号 用于自己发出的段的编号
			- 客户端和服务端生成各自的Seq初始随机值(ISN Inital Sequence Number)
			- Ack Acknowledge Number 确认号 用于确认收到的段的编号
			- 数据传输过程中的Seq和Ack
			- ![image.png](../assets/image_1684305338436_0.png)
- ## 3、TCP四次挥手
  collapsed:: true
	- 第一次挥手 FIN (Seq=u Ack=m)
	- 第二次挥手 ACK (Seq=v Ack=u+1)
	- 第三次挥手 FIN (Seq=w Ack=u+1)
	- 第四次挥手 ACK (Seq=u+1 Ack=w+1)
	- ![image.png](../assets/image_1684305375572_0.png){:height 796, :width 747}
	- 1. Wireshark 抓包查看挥手过程
		- ![image.png](../assets/image_1684305399081_0.png)
- ## 4、TLS握手(RSA)
  collapsed:: true
	- TCP三次握手
	  Client Hello（TLS版本 随机数 支持的加密套件）
	  ACK
	  Server Hello (TLS版本 随机数 选择加密套件)
	  Certificate （证书下发）
	  Server Hello Done
	  ACK
	  证书校验通过
	  Client Key Exchange (preMaster密钥)
	  Change Cipher Spec (切换加密传输)
	  Finished (所有握手数据加密)
	  ACK
	  Change Cipher Spec（切换加密传输）
	  Finished （所有握手数据加密）
	  ACK
	- ![image.png](../assets/image_1684305573863_0.png){:height 1196, :width 747}
	- 1. Wireshark 抓包查看握手过程
		- ![image.png](../assets/image_1684305594273_0.png)
- ## 5、TLS握手(ECDHE)
  collapsed:: true
	- TCP三次握手
	  Client Hello (TLS版本 随机数 支持的密码套件)
	  ACK
	  Server Hello (TLS版本 随机数 选择密码套件)
	  Certificate （证书下发）
	  Server Key Exchange (ECDH算法参数 公钥+签名)
	  ACK
	  证书校验通过
	  Client Key Exchange（ECDH算法参数 公钥）
	  Change Cypher Spec (切换加密传输)
	  Finished (所有握手数据加密)
	  ACK
	  Change Cypher Spec (切换加密传输)
	  Finished （所有握手数据加密）
	  ACK
	- ![image.png](../assets/image_1684305619450_0.png)
		- 1. Wireshark 抓包查看握手过程
			- ![image.png](../assets/image_1684305640169_0.png)
	-
- 6、参考
	-