- >保证一个Activity只有一个空白的SupportRequestManagerFragment。一个空白Fragment只有一个RequestManager
- ## RequestManagerRetriever类，看处理Fragment逻辑
  collapsed:: true
	- ```java
	   // 1、从 FragmentManager 中获取 SupportRequestManagerFragment
	      @NonNull
	      private SupportRequestManagerFragment getSupportRequestManagerFragment(
	              @NonNull final FragmentManager fm) {
	          // 先从FragmentManager 根据Tag获取一遍
	          SupportRequestManagerFragment current =
	                  (SupportRequestManagerFragment) fm.findFragmentByTag(FRAGMENT_TAG);
	          // 为空的话
	          if (current == null) {
	  
	              //  1.2 尝试从【记录保存】中获取 空白Fragment
	              current = pendingSupportRequestManagerFragments.get(fm);
	  
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
	          }
	          return current;
	      }
	  ```
	- ## handleMessage,移除map缓存里的空白Fragment
	  collapsed:: true
		- ```java
		      @Override
		      public boolean handleMessage(Message message) {
		  
		          switch (message.what) {
		              case ID_REMOVE_FRAGMENT_MANAGER:  // 移除 【记录保存】  1.3.5 post 一个消息
		                  android.app.FragmentManager fm = (android.app.FragmentManager) message.obj;
		                 pendingRequestManagerFragments.remove(fm); // 1.3.6 移除临时记录中的映射关系
		                  break;
		              case ID_REMOVE_SUPPORT_FRAGMENT_MANAGER: // 移除 【记录保存】  1.3.5 post 一个消息
		                  FragmentManager supportFm = (FragmentManager) message.obj;
		                  pendingSupportRequestManagerFragments.remove(supportFm); // 1.3.6 移除临时记录中的映射关系
		                  break;
		              default:
		                  break;
		          }
		  
		          return false;
		      }
		  ```
- # 逻辑总结，[[怎么确保一个Activity只有一个Fragment]]