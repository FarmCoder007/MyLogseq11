- Activity里的
	- ```java
	      @Nullable
	      public Object getLastNonConfigurationInstance() {
	          return mLastNonConfigurationInstances != null
	                  ? mLastNonConfigurationInstances.activity : null;
	      }
	  ```
	- 不为null  返回mLastNonConfigurationInstances.activity
	- 否则 返回null
-
- > 第一次获取肯定 为null