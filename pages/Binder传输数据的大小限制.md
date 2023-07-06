- ## 1、普通的由Zygote孵化而来的用户进程
	- 所映射的Binder内存大小是不到1M的，准确说是 1*1024*1024) - (4096 *2) （1M -8k）
	- 这个限制定义在ProcessState类中
		- ```c
		  #define BINDER_VM_SIZE ((1*1024*1024) - (4096 *2))
		  ```
	- （Intent也是不能传输大于1M的数据）不能传输大图bitmap
-
- ## 2、ServiceManager进程，它为自己申请的Binder内核空间是128K
	- 这个同ServiceManager的用途是分不开的，ServcieManager主要面向系统Service，只是简单的提供一些addServcie，getService的功能，不涉及多大的数据传输，因此不需要申请多大的内存：
- ## 3、内核层mmap开辟空间也有限制不超过4M
	- 不过由于APP中已经限制了不到1M，这里的限制似乎也没多大用途：