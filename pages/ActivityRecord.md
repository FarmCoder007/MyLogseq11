- # 概念
	- 它表示正在运行或待启动的**==Activity的记录或描述==**
	- 每当应用程序启动一个Activity，系统会创建一个对应的"ActivityRecord"对象来跟踪该Activity的状态、位置和其他相关信息。这个对象保存了Activity的各种属性，例如包名、类名、任务(Task)、进程(Process)、窗口(Window)等
- # 一、简介
	- ActivityRecord，源码中的注释介绍：An entry in the history stack, representing an activity. 翻译：历史栈中的一个条目，代表一个activity。
	- ```java
	  /**
	  * An entry in the history stack, representing an activity.
	  */
	  final class ActivityRecord extends ConfigurationContainer implements AppWindowContainerListener {
	      final ActivityManagerService service; // owner
	      final IApplicationToken.Stub appToken; // window manager token
	      AppWindowContainerController mWindowContainerController;
	      final ActivityInfo info; // all about me  Activity信息
	      final ApplicationInfo appInfo; // information about activity's app
	      //省略其他成员变量
	      //ActivityRecord所在的TaskRecord
	      private TaskRecord task; // the task this is in.
	      //构造方法，需要传递大量信息
	      ActivityRecord(ActivityManagerService _service, ProcessRecord _caller,
	                      int _launchedFromPid,
	                      int _launchedFromUid, String _launchedFromPackage, Intent
	                      _intent, String _resolvedType,
	                      ActivityInfo aInfo, Configuration _configuration,
	                                     com.android.server.am.ActivityRecord _resultTo, String
	                      _resultWho, int _reqCode,
	                      boolean _componentSpecified, boolean
	                      _rootVoiceInteraction,
	                      ActivityStackSupervisor supervisor, ActivityOptions
	                      options,
	      					com.android.server.am.ActivityRecord sourceRecord) {
	      }
	  }
	  ```
- # 二、ActivityRecord与TaskRecord建立联系
	- ActivityRecord中存在着大量的成员变量，包含了一个Activity的所有信息。 ActivityRecord中的成员变量task表示其所在的TaskRecord，由此可以看出：ActivityRecord与TaskRecord建立了联系
	- \frameworks\base\services\core\java\com\android\server\am\ActivityStarter.java
		- ```java
		  private int startActivity(IApplicationThread caller, Intent intent, Intent
		                            ephemeralIntent,
		                            String resolvedType, ActivityInfo aInfo, ResolveInfo rInfo,
		                            IVoiceInteractionSession voiceSession, IVoiceInteractor
		                            voiceInteractor,
		                            IBinder resultTo, String resultWho, int requestCode, int callingPid,
		                            int callingUid,
		                            String callingPackage, int realCallingPid, int realCallingUid, int
		                            startFlags,
		                            SafeActivityOptions options,
		                            boolean ignoreTargetSecurity, boolean componentSpecified,
		                            ActivityRecord[] outActivity,
		                            TaskRecord inTask, boolean
		                            allowPendingRemoteAnimationRegistryLookup) {
		  ActivityRecord r = new ActivityRecord(mService, callerApp, callingPid,
		                                    callingUid,
		                                    callingPackage, intent, resolvedType, aInfo,
		                                    mService.getGlobalConfiguration(),
		                                    resultRecord, resultWho, requestCode, componentSpecified,
		                                    voiceSession != null,
		                                    mSupervisor, checkedOptions, sourceRecord);
		  }
		  ```