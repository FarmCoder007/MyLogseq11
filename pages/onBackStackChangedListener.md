- ## 后退栈变动监听器
- 这个接口就是为我们提供一个 FragmentManager 所有 Fragment 生命周期变化的回调。
- 代码
	- ```java
	  //在 Fragment 回退栈中有变化时回调
	  public interface OnBackStackChangedListener {
	      public void onBackStackChanged();
	      }
	      //FragmentManager 中的 Fragment 生命周期监听
	      public abstract static class FragmentLifecycleCallbacks {
	      public void onFragmentPreAttached(FragmentManager fm, Fragment f,
	      											Context context) {}
	      public void onFragmentAttached(FragmentManager fm, Fragment f,
	      											Context context) {}
	      public void onFragmentCreated(FragmentManager fm, Fragment f,
	      Bundle savedInstanceState) {}
	      public void onFragmentActivityCreated(FragmentManager fm, Fragmentf,
	      								Bundle savedInstanceState) {}
	      public void onFragmentViewCreated(FragmentManager fm, Fragment f,
	      								View v, Bundle savedInstanceState) {}
	      public void onFragmentStarted(FragmentManager fm, Fragment f) {}
	      public void onFragmentResumed(FragmentManager fm, Fragment f) {}
	      public void onFragmentPaused(FragmentManager fm, Fragment f) {}
	      public void onFragmentStopped(FragmentManager fm, Fragment f) {}
	      public void onFragmentSaveInstanceState(FragmentManager fm,
	      Fragment f, Bundle outState) {}
	      public void onFragmentViewDestroyed(FragmentManager fm, Fragment f) {}
	      public void onFragmentDestroyed(FragmentManager fm, Fragment f) {}
	      public void onFragmentDetached(FragmentManager fm, Fragment f) {}
	      }
	  }
	  ```