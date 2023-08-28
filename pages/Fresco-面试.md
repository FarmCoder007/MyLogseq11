# 缓存，三级缓存
	- 已编码图片内存缓存：直接存的就是bitmap对象
	- 未编码图片内存缓存
	- 磁盘缓存
- # 3级缓存加载机制，按照优先级逐级命中并返回:也有Lrucache
	- 1）找Bitmap cache中是否有图片，有则加载，否则接着找；
	- 2）再看Encoded Memory中是否有图片，有则decode，transform 后加载图片，并将图片存储进Bitmap Cache,找不到则接着往下找;
	- 3)下一步看Disk Cache(官网有对这个命名的解释,看完大家都明白了：Yes, we know phones don’t have disks, but it’s too tedious to keep saying local storage cache)，有则decode，transform 后加载图片，并将图片存储进Bitmap Cache,Encoded Memory中，找不到则接着往下找;
	- 4）最后就是终极network cache寻找，有则decode，transform 后加载图片，并将图片存储进Bitmap Cache ,Encoded Memory, Disk Cache，当然如果没有就直接加载失败了。