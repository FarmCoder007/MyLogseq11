title:: ViewPager嵌套Listview. 自定义解决方法

- ## 背景：
	- 默认能上下滑动ListVIew,左右滑动viewPager。。这是因为google帮我们处理了
	- 如果我们自己处理事件冲突的话。如果让viewPager重写onIntercepedTouchEvent。
		- 拦截事件返回true。listView上下滑动不了,因为被父view拦截了，拿不到事件。
		- 如果返回false的话，listView消费了，viewPager又左右滑动不了。
- ## 方法1，内部拦截法，子view处理事件冲突
	- ## 1、让ListVIew，内部拦截法：子view处理事件冲突
		- 代码
		  collapsed:: true
			- ```java
			  public class MyListView extends ListView {
			  
			      public MyListView(Context context) {
			          super(context);
			      }
			  
			      public MyListView(Context context, AttributeSet attrs) {
			          super(context, attrs);
			      }
			  
			      // 内部拦截法：子view处理事件冲突
			      private int mLastX, mLastY;
			  
			      @Override
			      public boolean dispatchTouchEvent(MotionEvent event) {
			          int x = (int) event.getX();
			          int y = (int) event.getY();
			  
			          switch (event.getAction()) {
			              // down事件请求父view不拦截
			              case MotionEvent.ACTION_DOWN: {
			                  getParent().requestDisallowInterceptTouchEvent(true);
			                  break;
			              }
			              case MotionEvent.ACTION_MOVE: {
			                  int deltaX = x - mLastX;
			                  int deltaY = y - mLastY;
			                  if (Math.abs(deltaX) > Math.abs(deltaY)) {
			                      // move事件判断横向滑动 运行 viewPager 拦截
			                      getParent().requestDisallowInterceptTouchEvent(false);
			                  }
			                  break;
			              }
			              case MotionEvent.ACTION_UP: {
			                  break;
			  
			              }
			              default:
			                  break;
			          }
			  
			          mLastX = x;
			          mLastY = y;
			          return super.dispatchTouchEvent(event);
			      }
			  }
			  ```
		- 1、down事件请求父view不拦截
		- 2、move事件判断横向滑动 运行 viewPager 拦截
		- 但是光这样还不够。我们发现viewgroup的源码。disPacthTouchEvent里。Down事件会清除标记。上边请求父view不拦截的标记 会被清除那么 viewPager也要做下处理
	- ## 2、ViewPager，需设置onInterceptTouchEvent DOWN事件不拦截return false
		- 代码
			- ```java
			  public class BadViewPager extends ViewPager {
			  
			      private int mLastX, mLastY;
			  
			      public BadViewPager(@NonNull Context context) {
			          super(context);
			      }
			  
			      public BadViewPager(@NonNull Context context, @Nullable AttributeSet attrs) {
			          super(context, attrs);
			      }
			  
			  
			      @Override
			      public boolean onInterceptTouchEvent(MotionEvent event) {
			          if (event.getAction() == MotionEvent.ACTION_DOWN){
			              super.onInterceptTouchEvent(event);
			              return false;
			          }
			          return true;
			      }
			  }
			  
			  ```
		- 让viewpager Down时不拦截
- ## 方法2，外部拦截法：父容器处理冲突，我想要把事件分发给谁就分发给谁
	- ## 1、ListView不处理
	- ## 2、viewPager
		- ```java
		  package com.enjoy.leo_dispatch;
		  
		  import android.content.Context;
		  import android.support.annotation.NonNull;
		  import android.support.annotation.Nullable;
		  import android.support.v4.view.ViewPager;
		  import android.util.AttributeSet;
		  import android.util.Log;
		  import android.view.MotionEvent;
		  
		  public class BadViewPager extends ViewPager {
		  
		      private int mLastX, mLastY;
		  
		      public BadViewPager(@NonNull Context context) {
		          super(context);
		      }
		  
		      public BadViewPager(@NonNull Context context, @Nullable AttributeSet attrs) {
		          super(context, attrs);
		      }
		  
		      // 外部拦截法：父容器处理冲突
		      // 我想要把事件分发给谁就分发给谁
		      @Override
		      public boolean onInterceptTouchEvent(MotionEvent event) {
		          int x = (int) event.getX();
		          int y = (int) event.getY();
		  
		          switch (event.getAction()) {
		              case MotionEvent.ACTION_DOWN: {
		                  mLastX = (int) event.getX();
		                  mLastY = (int) event.getY();
		                  break;
		              }
		              case MotionEvent.ACTION_MOVE: {
		                  int deltaX = x - mLastX;
		                  int deltaY = y - mLastY;
		                  // 横向滑动的时候拦截
		                  if (Math.abs(deltaX) > Math.abs(deltaY)) {
		                      return true;
		                  }
		                  break;
		              }
		              case MotionEvent.ACTION_UP: {
		                  break;
		              }
		              default:
		                  break;
		          }
		  
		          return super.onInterceptTouchEvent(event);
		  
		      }
		  }
		  
		  ```