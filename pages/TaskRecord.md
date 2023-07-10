- # 一、概念
	- 用于表示[[#red]]==**任务（Task）的记录或描述**==，
	- 一个任务（Task）代表一组相关联的Activity的集合
	- TaskRecord，内部维护一个 ArrayList<ActivityRecord> 用来保存ActivityRecord（activity信息）。
- # 二、源码
	- \frameworks\base\services\core\java\com\android\server\am\TaskRecord.java
	- ```java
	  class TaskRecord extends ConfigurationContainer implements TaskWindowContainerListener {
	      final int taskId; //任务ID
	      final ArrayList<ActivityRecord> mActivities; //使用一个ArrayList来保存所有的 ActivityRecord
	      private ActivityStack mStack; //TaskRecord所在的ActivityStack
	        */
	      TaskRecord(ActivityManagerService service, int _taskId, Intent _intent,
	                    Intent _affinityIntent, String _affinity, String _rootAffinity,
	                    ComponentName _realActivity, ComponentName _origActivity, boolean
	                    _rootWasReset,
	                    boolean _autoRemoveRecents, boolean _askedCompatMode, int _userId,
	                    int _effectiveUid, String _lastDescription,
	                    ArrayList<ActivityRecord> activities,
	                    long lastTimeMoved, boolean neverRelinquishIdentity,
	                    TaskDescription _lastTaskDescription, int taskAffiliation, int
	                    prevTaskId,
	                    int nextTaskId, int taskAffiliationColor, int callingUid, String
	                    callingPackage,
	                    int resizeMode, boolean supportsPictureInPicture, boolean
	                    _realActivitySuspended,
	                    boolean userSetupComplete, int minWidth, int minHeight) {
	      }
	      //添加Activity到顶部
	      void addActivityToTop(com.android.server.am.ActivityRecord r) {
	          addActivityAtIndex(mActivities.size(), r);
	      }
	      //添加Activity到指定的索引位置
	      void addActivityAtIndex(int index, ActivityRecord r) {
	          //...
	          r.setTask(this);//为ActivityRecord设置TaskRecord，就是这里建立的联系
	          //...
	          index = Math.min(size, index);
	          mActivities.add(index, r);//添加到mActivities
	          //...
	      }
	  }
	  ```
	- 可以看到TaskRecord中使用了一个ArrayList来保存所有的ActivityRecord。 同样，TaskRecord中的mStack表示其所在的ActivityStack。 startActivity()时也会创建一个TaskRecord