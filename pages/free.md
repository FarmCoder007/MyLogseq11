- 主功能：查看可用内存，缺省单位KB。该命令比较简单、轻量，专注于查看剩余内存情况。数据来源
  于/proc/meminfo。
- 输出结果：
	- ```java
	  root@phone:/proc/sys/vm # free
	  total used free shared buffers
	  Mem: 2857032 1836040 1020992 0 75104
	  -/+ buffers: 1760936 1096096
	  Swap: 524284 0 524284
	  ```
- 对于Mem 行，存在的公式关系： total = used + free;
- 对于-/+ buffers 行： 1760936 = 1836040 - 75104(buffers); 1096096 = 1020992 +
  75104(buffers);