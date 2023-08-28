- 一个android应用需要为一个service单独开一个进程以完成与服务器交互的逻辑，同时在Application对象的onCreate方法中会执行对象的初始化操作，这时你会发现Application的oncreate方法会被调用两次，一些初始化的操作也会变成两次
- 问题原因：每个android应用都要运行在一个虚拟机上，当应用配置了两个进程时，其实是有两个虚拟机在运行，一个前台的应用进程，一个service后台进程，每个进程对应一个application对象，所以当应用配置了多个进程的时候，application对象的onCreate方法就会执行多次，所以如果在application的onCreate方法中开启轮询或者初始化大量数据时，其实是要做出区分的处理的；
- 解决问题：我们已经知道每个进程对应一个application对象，为了避免浪费资源，我们可以在application中通过进程的名称来区分具体应该加载哪些资源，执行哪些具体逻辑。
- ## 获取进程名
  collapsed:: true
	- ```java
	  private String getAppName(int pID) {
	          String processName = null;
	          ActivityManager am = (ActivityManager) this
	                  .getSystemService(ACTIVITY_SERVICE);
	          List l = am.getRunningAppProcesses();
	          Iterator i = l.iterator();
	          PackageManager pm = this.getPackageManager();
	          while (i.hasNext()) {
	              ActivityManager.RunningAppProcessInfo info = (ActivityManager.RunningAppProcessInfo) (i
	                      .next());
	              try {
	                  if (info.pid == pID) {
	                      CharSequence c = pm.getApplicationLabel(pm
	                              .getApplicationInfo(info.processName,
	                                      PackageManager.GET_META_DATA));
	                      processName = info.processName;
	                      return processName;
	                  }
	              } catch (Exception e) {
	              }
	          }
	          return processName;
	      }
	  ```
- ## 根据进程名进行一些初始化操作
	- ```java
	   int pid = android.os.Process.myPid();
	          String processAppName = getAppName(pid);
	          if (processAppName == null
	                  || !processAppName.equalsIgnoreCase("进程名")) {
	              // 则此application::onCreate 是被service 调用的，直接返回
	              return;
	          }
	  ```