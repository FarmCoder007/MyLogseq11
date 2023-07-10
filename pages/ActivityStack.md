- # 一、概念
	- 用于表示活动（Activity）的堆栈或堆叠结构。
	- 一个Activity栈有多个TaskRecord（任务描述），每个任务描述可对应多个ActivityRecord（activity信息）
	- ActivityStack,内部维护了一个 ArrayList<TaskRecord> ，用来管理`TaskRecord
- ```java
  class ActivityStack<T extends StackWindowController> extends ConfigurationContainer
  														implements StackWindowListener {
        /**
        * The back history of all previous (and possibly still
        * running) activities. It contains #TaskRecord objects.
        */
        //使用一个ArrayList来保存TaskRecord
        private final ArrayList<TaskRecord> mTaskHistory = new ArrayList<>();
        //持有一个ActivityStackSupervisor，所有的运行中的ActivityStacks都通过它来进行管里
        protected final ActivityStackSupervisor mStackSupervisor;
  
        ActivityStack(ActivityDisplay display, int stackId, ActivityStackSupervisor supervisor,
                                            int windowingMode, int activityType, boolean onTop) {
        }
        TaskRecord createTaskRecord(int taskId, ActivityInfo info, Intent intent,
                                              IVoiceInteractionSession voiceSession,
                                              IVoiceInteractor voiceInteractor,
                                              boolean toTop, int type) {
                    //创建一个task
                    TaskRecord task = new TaskRecord(mService, taskId, info, intent,
                    voiceSession, voiceInteractor, type);
                    //将task添加到ActivityStack中去
                    addTask(task, toTop, "createTaskRecord");
                    //其他代码略
                    return task;
        }
        //添加Task
        void addTask(final TaskRecord task, final boolean toTop, String reason)
        {
              addTask(task, toTop ? MAX_VALUE : 0, true /*
              schedulePictureInPictureModeChange */, reason);
              //其他代码略
        }
      //添加Task到指定位置
      void addTask(final TaskRecord task, int position, boolean schedulePictureInPictureModeChange,
                                                                                  String reason) {
            mTaskHistory.remove(task);//若存在，先移除
            //...
            mTaskHistory.add(position, task);//添加task到mTaskHistory
            task.setStack(this);//为TaskRecord设置ActivityStack
            //...
     }
  }
  ```
- 可以看到ActivityStack使用了一个ArrayList来保存TaskRecord。 另外，ActivityStack中还持有
- ActivityStackSupervisor对象，这个是用来管理ActivityStacks的。 ActivityStack是由
- ActivityStackSupervisor来创建的，实际ActivityStackSupervisor就是用来管理ActivityStack的