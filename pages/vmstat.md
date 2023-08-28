- 主功能：不仅可以查看内存情况，还可以查看进程运行队列、系统切换、CPU时间占比等情况，另外该
  指令还是周期性地动态输出。
- 用法：
	- ```java
	  Usage: vmstat [ -n iterations ] [ -d delay ] [ -r header_repeat ]
	  -n iterations 数据循环输出的次数
	  -d delay 两次数据间的延迟时长(单位：S)
	  -r header_repeat 循环多少次，再输出一次头信息行
	  ```
- 输入结果：
	- ```java
	  root@phone:/ # vmstat
	  procs memory system cpu
	  r b free mapped anon slab in cs flt us ni sy id wa ir
	  2 0 663436 232836 915192 113960 196 274 0 8 0 2 99 0 0
	  0 0 663444 232836 915108 113960 180 260 0 7 0 3 99 0 0
	  0 0 663476 232836 915216 113960 154 224 0 2 0 5 99 0 0
	  1 0 663132 232836 915304 113960 179 259 0 11 0 3 99 0 0
	  2 0 663124 232836 915096 113960 110 175 0 4 0 3 99 0 0
	  ```
- 参数列总共15个参数，分为4大类：
	- procs(进程)
		- r: Running队列中进程数量
		- b: IO wait的进程数量
	- memory(内存)
		- free: 可用内存大小
		- mapped：mmap映射的内存大小
		- anon: 匿名内存大小
		- slab: slab的内存大小
	- system(系统)
		- in: 每秒的中断次数(包括时钟中断)
		- cs: 每秒上下文切换的次数
	- cpu(处理器)
		- us: user time
		  ni: nice time
		  sy: system time
		  id: idle time
		  wa: iowait time
		  ir: interrupt time