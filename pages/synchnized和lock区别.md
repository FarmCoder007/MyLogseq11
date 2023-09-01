- 1、一个显示锁，一个隐式锁
- 2、lock是对象，sync是关键字
  id:: 64be3511-833c-4aa4-b138-9e6e96bad306
- 3、ReentrantLock 还提供了尝试拿锁的方法，，
  id:: 64be3521-0a8d-4b35-960a-c9f6af4efc7d
- 4、Synchronied为非公平锁
	- ReentrantLock 除了非公平锁 还提供了公平锁的实现
- 5、synchnized，锁上只能有一组wait notify
	- condition能有多组