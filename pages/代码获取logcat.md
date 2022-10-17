- [Android获取存储和打印输出Logcat日志](https://blog.csdn.net/weixin_45265547/article/details/121676425)
- ```java
      public static Runnable getLogInfoRunnable(ITestConsole console, String info, String level, String tag) {
          return new Runnable() {
              @RequiresApi(api = Build.VERSION_CODES.N)
              @Override
              public void run() {
                  console.appendMsg("*** " + info + " ***");
                  try {
                      //获取logcat日志信息
                      WLog.e("Wlog执行的命令：" + tag + ":" + level, info + ":" + level);
                      long startTime = System.currentTimeMillis();
                      Process mlogcatproc = Runtime.getRuntime().exec(new String[]{"logcat", "-e", "IWLog"});
                      Runtime.getRuntime().exec(new String[]{"logcat", "-c"});
                      long readTime = System.currentTimeMillis();
                      Log.e("Wlog","----Process执行时间："+(readTime - startTime)+"-----估计值："+mlogcatproc.getInputStream().available());
                      BufferedReader reader = new BufferedReader(new InputStreamReader(mlogcatproc.getInputStream()));
                      String line;
                      while ((line = reader.readLine()) != null) {
                          console.appendMsg("读到结果" + reader.readLine());
                      }
                      reader.close();
                      long readallTime = System.currentTimeMillis();
                      Log.e("Wlog","----Process执行时间："+(readTime - startTime)+"--读取时间："+(readallTime - readTime));
                  } catch (Exception e) {
                      console.appendMsg(e.getMessage());
                      e.printStackTrace();
                  }
                  console.appendMsg("----------------------------------------------------");
              }
          };
      }
  ```