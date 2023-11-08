# 一、基本使用
collapsed:: true
	- 存储
		- ```java
		  public void saveToSP(View view) {
		    // 第一个参数是sp的名字 xml配置文件的名字
		    // 第二个参数是保存时候用到的模式（常规(Context.MODE_PRIVATE):每次保存都会覆盖当前数据)
		    // 追加(Context.MODE_APPEND) 每次保存数据会追加到后面
		    SharedPreferences sp = getSharedPreferences("SP",
		                                                Context.MODE_PRIVATE);
		    sp.edit().putString("123","ljj").apply();
		  }
		  ```
	- 读取
		- ```java
		      public void getSpDATA(View view) {
		          SharedPreferences sp = getSharedPreferences("SP",
		                  Context.MODE_PRIVATE);
		          // 第2个参数：如果第一个参数key获取到的值是null，就用第二个参数的值代替
		          String value = sp.getString("123", "默认值");
		          Toast.makeText(this,""+value,Toast.LENGTH_SHORT).show();
		      }
		  ```
- # 二、数据的更新
  collapsed:: true
	- xml文件中的数据会缓存到内存的mMap中，每次在调用editor.putXXX()时，实际上会将新的数据存入在mMap，当调用commit()或apply()时，最终会将mMap的所有数据全量更新到xml文件里
- # 三、线程安全的
  collapsed:: true
	- 读操作
		- ```java
		   public String getString(String key, @Nullable String defValue) {
		        synchronized (mLock) {
		            String v = (String)mMap.get(key);
		            return v != null ? v : defValue;
		        }
		    }
		  ```
	- 写操作
		- >对于写操作而言，每次putXXX()并不能立即更新在mMap中 ,如果没有调用apply()方法，那么这些数据的更新理所当然应该被抛弃掉，但是如果直接更新在mMap中，那么数据就难以恢复。
		- >因此，Editor本身也应该持有一个mEditorMap对象，用于存储数据的更新；只有当调用apply()时，才尝试将mEditorMap与mMap进行合并，以达到数据更新的目的
		- ```java
		  public final class EditorImpl implements Editor {
		      @Override
		      public Editor putString(String key, String value) {
		          synchronized (mEditorLock) {
		              mEditorMap.put(key, value);
		              return this;
		          }
		  }
		  ```
		- 文件更新
			- ```java
			  // SharedPreferencesImpl.EditorImpl.enqueueDiskWrite()
			  synchronized (mWritingToDiskLock) {
			    writeToFile(mcr, isFromSyncCommit);
			  }
			  ```
- # 四、[[SP-ANR产生的原因]]
- # 参考
	- [源码分析](https://www.jianshu.com/p/768a91633fad)
	- [[Sp源码]]
	- [[Android 的 KV 存储库从 SharePreferences 到 MMKV 再到 DataStore]]
- # 面试
	- ## [[SP面试题]]