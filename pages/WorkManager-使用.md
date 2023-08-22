# 添加依赖
	- ```xml
	      // 2.4 最新的好像
	      // TODO 同学们，这里就是导入WorkManager的依赖支持，因为WorkManager是以扩展库的方式
	      def work_version = "2.3.4"
	      implementation "androidx.work:work-runtime:$work_version"
	  ```
- # 1、后台任务 并且 异步的
  collapsed:: true
	- ## MainWorker1
	  collapsed:: true
		- ```java
		  public class MainWorker1 extends Worker {
		  
		      private final static String TAG = MainWorker1.class.getSimpleName();
		  
		      public MainWorker1(@NonNull Context context, @NonNull WorkerParameters workerParams) {
		          super(context, workerParams);
		      }
		  
		      // 后台任务 并且 异步的
		      @NonNull
		      @Override
		      public Result doWork() {
		          Log.d(TAG, "MainWorker1 doWork: run started ... ");
		          try {
		              Thread.sleep(8000); // 睡眠
		          } catch (InterruptedException e) {
		              e.printStackTrace();
		              return Result.failure(); // 本次任务失败
		          } finally {
		              Log.d(TAG, "MainWorker1 doWork: run end ... ");
		          }
		          return Result.success(); // 本次任务成功
		      }
		  }
		  ```
	- ## 执行任务
	  collapsed:: true
		- ```java
		      /**
		       * 最简单的 执行任务
		       * 测试后台任务 1
		       * @param view
		       */
		      public void testBackgroundWork1(View view) {
		          OneTimeWorkRequest oneTimeWorkRequest =
		                  new OneTimeWorkRequest.Builder(MainWorker1.class).build();
		  
		          WorkManager.getInstance(this).enqueue(oneTimeWorkRequest);
		      }
		  ```
- # 2、work和Activity数据互相传递:借助LiveData
  collapsed:: true
	- ## MainWorker2
	  collapsed:: true
		- ```java
		  /**
		   * 数据 互相传递
		   * 后台任务
		   */
		  public class MainWorker2 extends Worker {
		  
		      public final static String TAG = MainWorker2.class.getSimpleName();
		  
		      private Context mContext;
		      private WorkerParameters workerParams;
		  
		      public MainWorker2(@NonNull Context context, @NonNull WorkerParameters workerParams) {
		          super(context, workerParams);
		          this.mContext = context;
		          this.workerParams = workerParams;
		      }
		  
		      @SuppressLint("RestrictedApi")
		      @NonNull
		      @Override
		      public Result doWork() {
		          Log.d(TAG, "MainWorker2 doWork: 后台任务执行了");
		  
		          // 接收 MainActivity传递过来的数据
		          String dataString = workerParams.getInputData().getString("Derry");
		          Log.d(TAG, "MainWorker2 doWork: 接收Activity传递过来的数据:" + dataString);
		  
		          // 反馈数据 给 MainActivity
		          // 把任务中的数据回传到activity中
		          Data outputData = new Data.Builder().putString("Derry", "三分归元气").build();
		          Result.Success success = new Result.Success(outputData);
		  
		          // return new Result.Failure(); // 本地执行 doWork 任务时 失败
		          // return new Result.Retry(); // 本地执行 doWork 任务时 重试一次
		          // return new Result.Success(); // 本地执行 doWork 任务时 成功 执行任务完毕
		          return success;
		      }
		  }
		  
		  ```
	- ## testBackgroundWork2
	  collapsed:: true
		- ```java
		   /**
		       * 数据 互相传递
		       * 测试后台任务 2
		       *
		       * @param view
		       */
		      public void testBackgroundWork2(View view) {
		          // 单一的任务  一次
		          OneTimeWorkRequest oneTimeWorkRequest1;
		  
		          // 数据
		          Data sendData = new Data.Builder().putString("Derry", "九阳神功").build();
		  
		          // 请求对象初始化
		          oneTimeWorkRequest1 = new OneTimeWorkRequest.Builder(MainWorker2.class)
		                  .setInputData(sendData) // 数据的携带
		                  .build();
		  
		          // 想接收 任务回馈的数据，需要状态机  LiveData(老师讲过 如果没有学过 观察者设计模式)
		          WorkManager.getInstance(this).getWorkInfoByIdLiveData(oneTimeWorkRequest1.getId())
		                  .observe(this, workInfo -> {
		  
		                      Log.d(MainWorker2.TAG, "状态：" + workInfo.getState().name());
		                      if (workInfo.getState().isFinished()) {
		  
		                          // 知道 本次任务执行的时候 各种状态  （SUCCEEDED，isFinished=true， 我再接收 任何回馈给我的数据）
		                          // 状态机 成功的时候 才去打印
		                          Log.d(MainWorker2.TAG, "Activity取到了任务回传的数据: " + workInfo.getOutputData().getString("Derry"));
		  
		                          Log.d(MainWorker2.TAG, "状态：isFinished=true 同学们注意：后台任务已经完成了...");
		                      }
		                  });
		  
		  
		          WorkManager.getInstance(this).enqueue(oneTimeWorkRequest1);
		      }
		  ```
- # 3、多个任务 顺序执行
  collapsed:: true
	- 1、
	  collapsed:: true
		- ```java
		  /**
		   * 后台任务3
		   */
		  public class MainWorker3 extends Worker {
		  
		      public final static String TAG = MainWorker3.class.getSimpleName();
		  
		      private Context mContext;
		      private WorkerParameters workerParams;
		  
		      // 有构造函数
		      public MainWorker3(@NonNull Context context, @NonNull WorkerParameters workerParams) {
		          super(context, workerParams);
		          this.mContext = context;
		          this.workerParams = workerParams;
		      }
		  
		      @SuppressLint("RestrictedApi")
		      @NonNull
		      @Override
		      public Result doWork() {
		  
		          Log.d(TAG, "MainWorker3 doWork: 后台任务执行了");
		  
		          return new Result.Success(); // 本地执行 doWork 任务时 成功 执行任务完毕
		      }
		  }
		  ```
	- 2、
	  collapsed:: true
		- ```java
		  /**
		   * 后台任务4
		   */
		  public class MainWorker4 extends Worker {
		  
		      public final static String TAG = MainWorker4.class.getSimpleName();
		  
		      private Context mContext;
		      private WorkerParameters workerParams;
		  
		      // 有构造函数
		      public MainWorker4(@NonNull Context context, @NonNull WorkerParameters workerParams) {
		          super(context, workerParams);
		          this.mContext = context;
		          this.workerParams = workerParams;
		      }
		  
		      @SuppressLint("RestrictedApi")
		      @NonNull
		      @Override
		      public Result doWork() {
		  
		          Log.d(TAG, "MainWorker4 doWork: 后台任务执行了");
		  
		          return new Result.Success(); // 本地执行 doWork 任务时 成功 执行任务完毕
		      }
		  }
		  
		  ```
	- 3、
	  collapsed:: true
		- ```java
		  /**
		   * 后台任务5
		   */
		  public class MainWorker5 extends Worker {
		  
		      public final static String TAG = MainWorker5.class.getSimpleName();
		  
		      private Context mContext;
		      private WorkerParameters workerParams;
		  
		      // 有构造函数
		      public MainWorker5(@NonNull Context context, @NonNull WorkerParameters workerParams) {
		          super(context, workerParams);
		          this.mContext = context;
		          this.workerParams = workerParams;
		      }
		  
		      @SuppressLint("RestrictedApi")
		      @NonNull
		      @Override
		      public Result doWork() {
		  
		          Log.d(TAG, "MainWorker5 doWork: 后台任务执行了");
		  
		          return new Result.Success(); // 本地执行 doWork 任务时 成功 执行任务完毕
		      }
		  }
		  
		  ```
	- 4、
	  collapsed:: true
		- ```java
		  /**
		   * 后台任务6
		   */
		  public class MainWorker6 extends Worker {
		  
		      public final static String TAG = MainWorker6.class.getSimpleName();
		  
		      private Context mContext;
		      private WorkerParameters workerParams;
		  
		      // 有构造函数
		      public MainWorker6(@NonNull Context context, @NonNull WorkerParameters workerParams) {
		          super(context, workerParams);
		          this.mContext = context;
		          this.workerParams = workerParams;
		      }
		  
		      @SuppressLint("RestrictedApi")
		      @NonNull
		      @Override
		      public Result doWork() {
		  
		          Log.d(TAG, "MainWorker6 doWork: 后台任务执行了");
		  
		          return new Result.Success(); // 本地执行 doWork 任务时 成功 执行任务完毕
		      }
		  }
		  ```
	- ## 使用
		- ```java
		   /**
		       * 多个任务 顺序执行
		       * 测试后台任务 3
		       * @param view
		       */
		      public void testBackgroundWork3(View view) {
		          // 单一的任务  一次
		          OneTimeWorkRequest oneTimeWorkRequest3 = new OneTimeWorkRequest.Builder(MainWorker3.class).build();
		          OneTimeWorkRequest oneTimeWorkRequest4 = new OneTimeWorkRequest.Builder(MainWorker4.class).build();
		          OneTimeWorkRequest oneTimeWorkRequest5 = new OneTimeWorkRequest.Builder(MainWorker5.class).build();
		          OneTimeWorkRequest oneTimeWorkRequest6 = new OneTimeWorkRequest.Builder(MainWorker6.class).build();
		  
		          // 顺序执行 3 4 5 6
		          WorkManager.getInstance(this)
		                  .beginWith(oneTimeWorkRequest3)  // 检查的任务
		                  .then(oneTimeWorkRequest4) // 业务1任务
		                  .then(oneTimeWorkRequest5) // 业务2任务
		                  .then(oneTimeWorkRequest6) // 最后检查工作任务
		                  .enqueue();
		  
		          // 需求：先执行  3  4    最后执行 6
		          List<OneTimeWorkRequest> oneTimeWorkRequests = new ArrayList<>();
		          oneTimeWorkRequests.add(oneTimeWorkRequest3); // 先同步日志信息
		          oneTimeWorkRequests.add(oneTimeWorkRequest4); // 先更新服务器数据信息
		  
		          WorkManager.getInstance(this).beginWith(oneTimeWorkRequests)
		                  .then(oneTimeWorkRequest6) // 最后再 检查同步
		                  .enqueue();
		      }
		  ```
- # 4、周期性任务
  collapsed:: true
	- ```java
	      /**
	       * 重复执行后台任务  非单个任务，多个任务
	       * 测试后台任务 4
	       * @param view
	       */
	      public void testBackgroundWork4(View view) {
	          // OneTimeWorkRequest 单个
	  
	          // 重复的任务  多次/循环/轮询  , 哪怕设置为 10秒 轮询一次,   那么最少轮询/循环一次 15分钟（Google规定的）
	          // 不能小于15分钟，否则默认修改成 15分钟
	          PeriodicWorkRequest periodicWorkRequest
	                  = new PeriodicWorkRequest.Builder(MainWorker3.class, 10, TimeUnit.SECONDS).build();
	  
	          // 【状态机】  为什么一直都是 ENQUEUE，因为 你是轮询的任务，所以你看不到 SUCCESS     [如果你是单个任务，就会看到SUCCESS]
	          // 监听状态
	          WorkManager.getInstance(this).getWorkInfoByIdLiveData(periodicWorkRequest.getId())
	                  .observe(this, new Observer<WorkInfo>() {
	                      @Override
	                      public void onChanged(WorkInfo workInfo) {
	                          Log.d(MainWorker2.TAG, "状态：" + workInfo.getState().name()); // ENQUEEN   SUCCESS
	                          if (workInfo.getState().isFinished()) {
	                              Log.d(MainWorker2.TAG, "状态：isFinished=true 同学们注意：后台任务已经完成了...");
	                          }
	                      }
	                  });
	  
	          WorkManager.getInstance(this).enqueue(periodicWorkRequest);
	  
	          // 取消 任务的执行
	          // WorkManager.getInstance(this).cancelWorkById(periodicWorkRequest.getId());
	      }
	  
	  ```
- # 5、约束性任务
  collapsed:: true
	- ```java
	   /**
	       * 约束条件，约束后台任务执行
	       * 测试后台任务 5
	       * @param view
	       */
	      @RequiresApi(api = Build.VERSION_CODES.M)
	      public void testBackgroundWork5(View view) {
	          // 约束条件，必须满足我的条件，才能执行后台任务 （在连接网络，插入电源 并且 处于空闲时）
	          Constraints constraints = new Constraints.Builder()
	                  .setRequiredNetworkType(NetworkType.CONNECTED) // 网络链接中...
	                  /*.setRequiresCharging(true) // 充电中..
	                  .setRequiresDeviceIdle(true) // 空闲时.. (例如：你没有玩游戏，你没有看片)*/
	                  .build();
	  
	          /**
	           * 除了上面设置的约束外，WorkManger还提供了以下的约束作为Work执行的条件：
	           *  setRequiredNetworkType：网络连接设置
	           *  setRequiresBatteryNotLow：是否为低电量时运行 默认false
	           *  setRequiresCharging：是否要插入设备（接入电源），默认false
	           *  setRequiresDeviceIdle：设备是否为空闲，默认false
	           *  setRequiresStorageNotLow：设备可用存储是否不低于临界阈值
	           */
	  
	          // 请求对象
	          OneTimeWorkRequest request = new OneTimeWorkRequest.Builder(MainWorker3.class)
	                  .setConstraints(constraints) // Request关联 约束条件
	                  .build();
	  
	          // 加入队列
	          WorkManager.getInstance(this).enqueue(request);
	      }
	  
	  ```
- # 6、app杀死也能执行
  collapsed:: true
	- ## 任务
		- ```java
		  /**
		   * （你怎么知道，他被杀掉后，还在后台执行？）写入文件的方式，向同学们证明 Derry说的 所言非虚
		   * 后台任务7
		   */
		  public class MainWorker7 extends Worker {
		  
		      public final static String TAG = MainWorker7.class.getSimpleName();
		  
		      public static final String SP_NAME = "spNAME"; // SP name
		      public static final String SP_KEY = "spKEY"; // KEY
		  
		  
		      private Context mContext;
		      private WorkerParameters workerParams;
		  
		      // 有构造函数
		      public MainWorker7(@NonNull Context context, @NonNull WorkerParameters workerParams) {
		          super(context, workerParams);
		          this.mContext = context;
		          this.workerParams = workerParams;
		      }
		  
		      @SuppressLint("RestrictedApi")
		      @NonNull
		      @Override
		      public Result doWork() {
		          Log.d(TAG, "MainWorker7 doWork: 后台任务执行了 started");
		  
		          try {
		              Thread.sleep(8000);
		          } catch (InterruptedException e) {
		              e.printStackTrace();
		          }
		  
		          // 获取SP
		          SharedPreferences sp = getApplicationContext().getSharedPreferences(SP_NAME, Context.MODE_PRIVATE);
		  
		          // 获取 sp 里面的值
		          int spIntValue = sp.getInt(SP_KEY, 0);
		  
		          sp.edit().putInt(SP_KEY, ++spIntValue).apply();
		  
		          Log.d(TAG, "MainWorker7 doWork: 后台任务执行了 end");
		  
		          return new Result.Success(); // 本地执行 doWork 任务时 成功 执行任务完毕
		      }
		  
		  
		  }
		  
		  ```
	- ## 使用
	  collapsed:: true
		- ```java
		  
		  public class MainActivity extends AppCompatActivity implements SharedPreferences.OnSharedPreferenceChangeListener {
		  
		      @Override
		      protected void onCreate(Bundle savedInstanceState) {
		          super.onCreate(savedInstanceState);
		          setContentView(R.layout.activity_main);
		          bt6 = findViewById(R.id.bt6);
		  
		          // 绑定监听
		          SharedPreferences sp = getSharedPreferences(MainWorker7.SP_NAME, Context.MODE_PRIVATE);
		          sp.registerOnSharedPreferenceChangeListener(this); // 给SP绑定监听
		          updateToUI(); // 第一次初始化一把
		      }
		  
		  /**
		       * （你怎么知道，他被杀掉后，还在后台执行？）写入文件的方式（SP），向同学们证明 Derry说的 所言非虚
		       * 测试后台任务 6
		       * @param view
		       */
		      public void testBackgroundWork6(View view) {
		          // 约束条件
		          Constraints constraints = new Constraints.Builder()
		                  .setRequiredNetworkType(NetworkType.CONNECTED) // 约束条件，必须是网络连接
		                  .build();
		  
		          // 构建Request
		          OneTimeWorkRequest request = new OneTimeWorkRequest.Builder(MainWorker7.class)
		                  .setConstraints(constraints)
		                  .build();
		  
		          // 加入队列
		          WorkManager.getInstance(this).enqueue(request);
		      }
		   // 从SP里面获取值，显示到界面给用户看就行了
		      private final void updateToUI() {
		          SharedPreferences sp = getSharedPreferences(MainWorker7.SP_NAME, Context.MODE_PRIVATE);
		          int resultValue = sp.getInt(MainWorker7.SP_KEY, 0);
		          bt6.setText("测试后台任务六 -- " + resultValue);
		      }
		  
		      // 监听 SP 里面数据的变化，你只要敢变，我这里就知道啦
		      @Override
		      public void onSharedPreferenceChanged(SharedPreferences sharedPreferences, String key) {
		          updateToUI();
		      }
		  
		      // SP归零
		      public void spReset(View view) {
		          SharedPreferences sp = getSharedPreferences(MainWorker7.SP_NAME, Context.MODE_PRIVATE);
		          sp.edit().putInt(MainWorker7.SP_KEY, 0).apply();
		          updateToUI();
		      }
		  }
		  ```
	-
		-