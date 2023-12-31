- 问题：
  collapsed:: true
	- 在android 9.0系统上如果多个进程使用WebView需要使用官方提供的api在子进程中给webview的数据文件夹设置后缀：
	- ```
	  WebView.setDataDirectorySuffix(suffix);
	  ```
	- 否则出现崩溃
		- ```
		  Using WebView from more than one process at once with the same data directory is not supported. https://crbug.com/558377
		  
		  1 com.android.webview.chromium.WebViewChromiumAwInit.startChromiumLocked(WebViewChromiumAwInit.java:63)
		  2 com.android.webview.chromium.WebViewChromiumAwInitForP.startChromiumLocked(WebViewChromiumAwInitForP.java:3)
		  3 com.android.webview.chromium.WebViewChromiumAwInit$3.run(WebViewChromiumAwInit.java:3)
		  4 android.os.Handler.handleCallback(Handler.java:873)
		  5 android.os.Handler.dispatchMessage(Handler.java:99)
		  6 android.os.Looper.loop(Looper.java:220)
		  7 android.app.ActivityThread.main(ActivityThread.java:7437)
		  8 java.lang.reflect.Method.invoke(Native Method)
		  9 com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:500)
		  10 com.android.internal.os.ZygoteInit.main(ZygoteInit.java:865)
		  ```
	- 通过使用官方提供的方法后问题只减少了一部分，从bugly后台依然能收到此问题的大量崩溃信息，以至于都冲上了崩溃问题Top3。
- 问题分析：
  collapsed:: true
	- ```java
	  从源码分析调用链最终调用到了AwDataDirLock类中的lock方法。
	  
	  public class WebViewChromiumAwInit {
	      protected void startChromiumLocked() {
	              ...
	              AwBrowserProcess.start();
	              ... 
	      }
	  }
	  public final class AwBrowserProcess {
	      public static void start() {
	              ...
	              AwDataDirLock.lock(appContext);
	  }
	  
	  abstract class AwDataDirLock {
	      private static final String TAG = "AwDataDirLock";
	      private static final String EXCLUSIVE_LOCK_FILE = "webview_data.lock";
	      // This results in a maximum wait time of 1.5s
	      private static final int LOCK_RETRIES = 16;
	      private static final int LOCK_SLEEP_MS = 100;
	      private static RandomAccessFile sLockFile;
	      private static FileLock sExclusiveFileLock;
	  
	      static void lock(final Context appContext) {
	          try (ScopedSysTraceEvent e1 = ScopedSysTraceEvent.scoped("AwDataDirLock.lock");
	               StrictModeContext ignored = StrictModeContext.allowDiskWrites()) {
	              if (sExclusiveFileLock != null) {
	                  // We have already called lock() and successfully acquired the lock in this process.
	                  // This shouldn't happen, but is likely to be the result of an app catching an
	                  // exception thrown during initialization and discarding it, causing us to later
	                  // attempt to initialize WebView again. There's no real advantage to failing the
	                  // locking code when this happens; we may as well count this as the lock being
	                  // acquired and let init continue (though the app may experience other problems
	                  // later).
	                  return;
	              }
	              // If we already called lock() but didn't succeed in getting the lock, it's possible the
	              // app caught the exception and tried again later. As above, there's no real advantage
	              // to failing here, so only open the lock file if we didn't already open it before.
	              if (sLockFile == null) {
	                  String dataPath = PathUtils.getDataDirectory();
	                  File lockFile = new File(dataPath, EXCLUSIVE_LOCK_FILE);
	                  try {
	              // Note that the file is kept open intentionally.
	                      sLockFile = new RandomAccessFile(lockFile, "rw");
	                  } catch (IOException e) {
	                  // Failing to create the lock file is always fatal; even if multiple processes
	                  // are using the same data directory we should always be able to access the file
	                  // itself.
	                      throw new RuntimeException("Failed to create lock file " + lockFile, e);
	                  }
	              }
	              // Android versions before 11 have edge cases where a new instance of an app process can
	              // be started while an existing one is still in the process of being killed. This can
	              // still happen on Android 11+ because the platform has a timeout for waiting, but it's
	              // much less likely. Retry the lock a few times to give the old process time to fully go
	              // away.
	              for (int attempts = 1; attempts <= LOCK_RETRIES; ++attempts) {
	                  try {
	                      sExclusiveFileLock = sLockFile.getChannel().tryLock();
	                  } catch (IOException e) {
	                  // Older versions of Android incorrectly throw IOException when the flock()
	                  // call fails with EAGAIN, instead of returning null. Just ignore it.
	                  }
	                  if (sExclusiveFileLock != null) {
	                      // We got the lock; write out info for debugging.
	                      writeCurrentProcessInfo(sLockFile);
	                      return;
	                  }
	                  // If we're not out of retries, sleep and try again.
	                  if (attempts == LOCK_RETRIES) break;
	                  try {
	                      Thread.sleep(LOCK_SLEEP_MS);
	                  } catch (InterruptedException e) {
	                  }
	              }
	              // We failed to get the lock even after retrying.
	              // Many existing apps rely on this even though it's known to be unsafe.
	              // Make it fatal when on P for apps that target P or higher
	              String error = getLockFailureReason(sLockFile);
	              boolean dieOnFailure = Build.VERSION.SDK_INT >= Build.VERSION_CODES.P
	                      && appContext.getApplicationInfo().targetSdkVersion >= Build.VERSION_CODES.P;
	              if (dieOnFailure) {
	                  throw new RuntimeException(error);
	              } else {
	                  Log.w(TAG, error);
	              }
	          }
	      }
	  
	      private static void writeCurrentProcessInfo(final RandomAccessFile file) {
	          try {
	              // Truncate the file first to get rid of old data.
	              file.setLength(0);
	              file.writeInt(Process.myPid());
	              file.writeUTF(ContextUtils.getProcessName());
	          } catch (IOException e) {
	              // Don't crash just because something failed here, as it's only for debugging.
	              Log.w(TAG, "Failed to write info to lock file", e);
	          }
	      }
	  
	      private static String getLockFailureReason(final RandomAccessFile file) {
	          final StringBuilder error = new StringBuilder("Using WebView from more than one process at "
	                  + "once with the same data directory is not supported. https://crbug.com/558377 "
	                  + ": Current process ");
	          error.append(ContextUtils.getProcessName());
	          error.append(" (pid ").append(Process.myPid()).append("), lock owner ");
	          try {
	              int pid = file.readInt();
	              String processName = file.readUTF();
	              error.append(processName).append(" (pid ").append(pid).append(")");
	              // Check the status of the pid holding the lock by sending it a null signal.
	              // This doesn't actually send a signal, just runs the kernel access checks.
	              try {
	                  Os.kill(pid, 0);
	                  // No exception means the process exists and has the same uid as us, so is
	                  // probably an instance of the same app. Leave the message alone.
	              } catch (ErrnoException e) {
	                  if (e.errno == OsConstants.ESRCH) {
	                      // pid did not exist - the lock should have been released by the kernel,
	                      // so this process info is probably wrong.
	                      error.append(" doesn't exist!");
	                  } else if (e.errno == OsConstants.EPERM) {
	                      // pid existed but didn't have the same uid as us.
	                      // Most likely the pid has just been recycled for a new process
	                      error.append(" pid has been reused!");
	                  } else {
	                      // EINVAL is the only other documented return value for kill(2) and should never
	                      // happen for signal 0, so just complain generally.
	                      error.append(" status unknown!");
	                  }
	              }
	          } catch (IOException e) {
	              // We'll get IOException if we failed to read the pid and process name; e.g. if the
	              // lockfile is from an old version of WebView or an IO error occurred somewhere.
	              error.append(" unknown");
	          }
	          return error.toString();
	      }
	  }
	  ```
	- lock方法会对webview数据目录中的webview_data.lock文件在for循环中尝试加锁16次，注释中也说明了这么做的原因：可能出现的极端情况是一个旧进程正在被杀死时一个新的进程启动了，看来Google工程师对这个问题也很头痛；如果加锁成功会将该进程id和进程名写入到文件，如果加锁失败则会抛出异常。所以在android9.0以上检测应用是否存在多进程共用WebView数据目录的原理就是进程持有WebView数据目录中的webview_data.lock文件的锁。所以如果子进程也对相同文件尝试加锁则会导致应用崩溃
- 解决：
  collapsed:: true
	- 目前大部分手机会在应用崩溃时自动重启应用，猜测当手机系统运行较慢时这时就会出现注释中提到的当一个旧进程正在被杀死时一个新的进程启动了的情况。既然获取文件锁失败就会发生崩溃，并且该文件只是用于加锁判断是否存在多进程共用WebView数据目录，每次加锁成功都会重新写入对应进程信息，那么我们可以在应用启动时对该文件尝试加锁，如果加锁失败就删除该文件并重新创建，加锁成功就立即释放锁，这样当系统尝试加锁时理论上是可以加锁成功的，也就避免了这个问题的发生。
	- ```java
	  private static void handleWebviewDir(Context context) {
	  if (Build.VERSION.SDK_INT < Build.VERSION_CODES.P) {
	              return;
	          }
	          try {
	              String suffix = "";
	              String processName = getProcessName(context);
	              if (!TextUtils.equals(context.getPackageName(), processName)) {//判断不等于默认进程名称
	                  suffix = TextUtils.isEmpty(processName) ? context.getPackageName() : processName;
	                  WebView.setDataDirectorySuffix(suffix);
	                  suffix = "_" + suffix;
	              }
	              tryLockOrRecreateFile(context,suffix);
	          } catch (Exception e) {
	              e.printStackTrace();
	          }
	      }
	  
	      @TargetApi(Build.VERSION_CODES.P)
	      private static void tryLockOrRecreateFile(Context context,String suffix) {
	          String sb = context.getDataDir().getAbsolutePath() +
	                  "/app_webview"+suffix+"/webview_data.lock";
	          File file = new File(sb);
	          if (file.exists()) {
	              try {
	                  FileLock tryLock = new RandomAccessFile(file, "rw").getChannel().tryLock();
	                  if (tryLock != null) {
	                      tryLock.close();
	                  } else {
	                      createFile(file, file.delete());
	                  }
	              } catch (Exception e) {
	                  e.printStackTrace();
	                  boolean deleted = false;
	                  if (file.exists()) {
	                      deleted = file.delete();
	                  }
	                  createFile(file, deleted);
	              }
	          }
	      }
	  
	      private static void createFile(File file, boolean deleted){
	          try {
	              if (deleted && !file.exists()) {
	                  file.createNewFile();
	              }
	          } catch (Exception e) {
	              e.printStackTrace();
	          }
	      }
	  ```