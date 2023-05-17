- ##  1. 网络分层
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
	- 第一次握手: SYN(Seq=x Ack=0)
	  第二次握手: SYN+ACK(Seq=y Ack=x+1)
	  第三次握手: ACK(Seq=x+1 Ack=y+1)
-