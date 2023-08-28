- 功能：能否查看更加详细的内存信息
- 指令
  collapsed:: true
	- ```java
	  cat /proc/meminfo
	  ```
- 输出结果如下(结果内存值不带小数点，此处添加小数点的目的是为了便于比对大小)
  collapsed:: true
	- ```java
	  root@phone:/ # cat /proc/meminfo
	  MemTotal: 2857.032 kB //RAM可用的总大小 (即物理总内存减去系统预留和内核二进
	  制代码大小)
	  MemFree: 1020.708 kB //RAM未使用的大小
	  Buffers: 75.104 kB //用于文件缓冲
	  Cached: 448.244 kB //用于高速缓存
	  SwapCached: 0 kB //用于swap缓存
	  Active: 832.900 kB //活跃使用状态，记录最近使用过的内存，通常不回收用于其
	  它目的
	  Inactive: 391.128 kB //非活跃使用状态，记录最近并没有使用过的内存，能够被回
	  收用于其他目的
	  Active(anon): 700.744 kB //Active = Active(anon) + Active(file)
	  Inactive(anon): 228 kB //Inactive = Inactive(anon) + Inactive(file)
	  Active(file): 132.156 kB
	  Inactive(file): 390.900 kB
	  Unevictable: 0 kB
	  Mlocked: 0 kB
	  SwapTotal: 524.284 kB //swap总大小
	  SwapFree: 524.284 kB //swap可用大小
	  Dirty: 0 kB //等待往磁盘回写的大小
	  Writeback: 0 kB //正在往磁盘回写的大小
	  AnonPages: 700.700 kB //匿名页，用户空间的页表，没有对应的文件
	  Mapped: 187.096 kB //文件通过mmap分配的内存，用于map设备、文件或者库
	  Shmem: .312 kB
	  Slab: 91.276 kB //kernel数据结构的缓存大小，
	  Slab=SReclaimable+SUnreclaim
	  SReclaimable: 32.484 kB //可回收的slab的大小
	  SUnreclaim: 58.792 kB //不可回收slab的大小
	  KernelStack: 25.024 kB
	  PageTables: 23.752 kB //以最低的页表级
	  NFS_Unstable: 0 kB //不稳定页表的大小
	  Bounce: 0 kB
	  WritebackTmp: 0 kB
	  CommitLimit: 1952.800 kB
	  Committed_AS: 82204.348 kB //评估完成的工作量，代表最糟糕case下的值，该值也包含
	  swap内存
	  VmallocTotal: 251658.176 kB //总分配的虚拟地址空间
	  VmallocUsed: 166.648 kB //已使用的虚拟地址空间
	  VmallocChunk: 251398.700 kB //虚拟地址空间可用的最大连续内存块
	  ```
- 对于cache和buffer也是系统可以使用的内存。所以系统总的可用内存为 MemFree+Buffers+Cached