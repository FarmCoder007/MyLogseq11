## 1、SP的缺点
	- 1、不支持多进程。就是一个进程修改了，另一个进程拿不到最新的
	- 2、不支持增量更新
		- 1、当修改一个键值对时，提交到内存后
		- 2 它会把整个数据又重新写入到磁盘文件中，虽然我们只修改一个小点。一个键值对。
		- 3 这样就会导致不必要的内存开销，不支持增量数据写入。
	- 3、使用不当导致ANR
- ## 2、[[SP-ANR产生的原因]]
- ## 3、数据的更新
	- 每次在调用editor.putXXX()时，实际上会将新的数据缓存到内存的mMap中，当调用commit()或apply()时，最终会将mMap的所有数据全量更新到xml文件里
- ## 4、线程安全的
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