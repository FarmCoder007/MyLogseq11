- 1、 根据Tag先从FragmentManager 获取一遍
	- ```java
	  SupportRequestManagerFragment current =
	                  (SupportRequestManagerFragment) fm.findFragmentByTag(FRAGMENT_TAG);
	  ```
- 2、没找到从缓存map去找[[#red]]==**（第一个保证）**==
	- ```java
	  
	  //  1.2 尝试从【记录保存】中获取 空白Fragment
	  current = pendingSupportRequestManagerFragments.get(fm);
	  ```
- 3、还是没有的话，去创建实例
	- ```java
	  // 1.3 实例化 Fragment
	  if (current == null) {
	  
	    // 1.3.1 创建对象 空白的Fragment
	    current = new SupportRequestManagerFragment();  // 重复创建
	  
	    // 1.3.2 【记录保存】映射关系 进行保存   第一个保障
	    // 一个MainActivity == 一个空白的SupportRequestManagerFragment == 一个RequestManager
	    pendingSupportRequestManagerFragments.put(fm, current);
	  
	    // 1.3.3 提交 Fragment 事务
	    fm.beginTransaction().add(current, FRAGMENT_TAG).commitAllowingStateLoss();
	  
	    // 1.3.4 post 一个消息，移除map缓存的空白Fragment
	    handler.obtainMessage(ID_REMOVE_SUPPORT_FRAGMENT_MANAGER, fm).sendToTarget();
	  }
	  ```
	- 1、创建空白Fragment实例
	- 2、记录映射关系到map里[[#red]]==**（第一个保证）**==
	  collapsed:: true
		- 因为即使添加多次，第一次没有存缓存，第二次就从map里取了。所以不会事务添加多次的
		- ```java
		  Glide.with(this).load("xx").into(ImageView(this))
		    Glide.with(this).load("xx").into(ImageView(this))
		    Glide.with(this).load("xx").into(ImageView(this))
		  ```
	- 3、提交事务commit
	- 4、发送handler消息来移除map里存的空白Fragment
	- > 这里为啥事务commit前先加到了map缓存。然后事务提交，再通过handler移除？
		- 为了确保只添加一个Fragment，事务是通过handler实现的。handler messagequeue是按时间顺序等待队列。
		- 事务commit，只是添加到等待队列里。handler发送一个消息，触发messageQueue轮转。让队列里的消息马上工作，这样就可以被处理到
		- 而且Fragment被添加上了。fm也就可以拿到了。第一次判断就可以 了。所以可以清空