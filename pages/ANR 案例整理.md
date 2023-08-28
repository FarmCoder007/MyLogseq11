# 一、主线程被其他线程lock，导致死锁（关键字held by）
	- ```java
	  waiting on <0x1cd570> (a android.os.MessageQueue)
	  DALVIK THREADS:
	  "main" prio=5 tid=3 TIMED_WAIT
	    | group="main" sCount=1 dsCount=0 s=0 obj=0x400143a8
	    | sysTid=691 nice=0 sched=0/0 handle=-1091117924
	    at java.lang.Object.wait(Native Method)
	  waiting on <0x1cd570> (a android.os.MessageQueue)
	  at java.lang.Object.wait(Object.java:195)
	  at android.os.MessageQueue.next(MessageQueue.java:144)
	  at android.os.Looper.loop(Looper.java:110)
	  at android.app.ActivityThread.main(ActivityThread.java:3742)
	  at java.lang.reflect.Method.invokeNative(Native Method)
	  at java.lang.reflect.Method.invoke(Method.java:515)
	  at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:739)
	  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:497)
	  at dalvik.system.NativeStart.main(Native Method)
	  
	  "Binder Thread #3" prio=5 tid=15 NATIVE
	  | group="main" sCount=1 dsCount=0 s=0 obj=0x434e7758
	  | sysTid=734 nice=0 sched=0/0 handle=1733632
	  at dalvik.system.NativeStart.run(Native Method)
	  
	  "Binder Thread #2" prio=5 tid=13 NATIVE
	  | group="main" sCount=1 dsCount=0 s=0 obj=0x1cd570
	  | sysTid=696 nice=0 sched=0/0 handle=1369840
	  at dalvik.system.NativeStart.run(Native Method)
	  
	  "Binder Thread #1" prio=5 tid=11 NATIVE
	  | group="main" sCount=1 dsCount=0 s=0 obj=0x433aca10
	  | sysTid=695 nice=0 sched=0/0 handle=1367448
	  at dalvik.system.NativeStart.run(Native Method)
	  
	  ----- end 691 -----
	  ```
- # 二、主线程做耗时的操作：比如数据库读写。
  collapsed:: true
	- ```java
	  "main" prio=5 tid=1 Native
	  held mutexes=
	  kernel: (couldn't read /proc/self/task/11003/stack)
	  native: #00 pc 000492a4 /system/lib/libc.so (nanosleep+12)
	  native: #01 pc 0002dc21 /system/lib/libc.so (usleep+52)
	  native: #02 pc 00009cab /system/lib/libsqlite.so (???)
	  native: #03 pc 00011119 /system/lib/libsqlite.so (???)
	  native: #04 pc 00016455 /system/lib/libsqlite.so (???)
	  native: #16 pc 0000fa29 /system/lib/libsqlite.so (???)
	  native: #17 pc 0000fad7 /system/lib/libsqlite.so (sqlite3_prepare16_v2+14)
	  native: #18 pc 0007f671 /system/lib/libandroid_runtime.so (???)
	  native: #19 pc 002b4721 /system/framework/arm/boot-framework.oat (Java_android_database_sqlite_SQLiteConnection_nativePrepareStatement__JLjava_lang_String_2+116)
	  at android.database.sqlite.SQLiteConnection.setWalModeFromConfiguration(SQLiteConnection.java:294)
	  at android.database.sqlite.SQLiteConnection.open(SQLiteConnection.java:215)
	  at android.database.sqlite.SQLiteConnection.open(SQLiteConnection.java:193)
	  at android.database.sqlite.SQLiteConnectionPool.openConnectionLocked(SQLiteConnectionPool.java:463)
	  at android.database.sqlite.SQLiteConnectionPool.open(SQLiteConnectionPool.java:185)
	  at android.database.sqlite.SQLiteConnectionPool.open(SQLiteConnectionPool.java:177)
	  at android.database.sqlite.SQLiteDatabase.openInner(SQLiteDatabase.java:808)
	  locked <0x0db193bf> (a java.lang.Object)
	  at android.database.sqlite.SQLiteDatabase.open(SQLiteDatabase.java:793)
	  at android.database.sqlite.SQLiteDatabase.openDatabase(SQLiteDatabase.java:696)
	  at android.app.ContextImpl.openOrCreateDatabase(ContextImpl.java:690)
	  at android.content.ContextWrapper.openOrCreateDatabase(ContextWrapper.java:299)
	  at android.database.sqlite.SQLiteOpenHelper.getDatabaseLocked(SQLiteOpenHelper.java:223)
	  at android.database.sqlite.SQLiteOpenHelper.getWritableDatabase(SQLiteOpenHelper.java:163)
	  locked <0x045a4a8c> (a com.xxxx.video.common.data.DataBaseHelper)
	  at com.xxxx.video.common.data.DataBaseORM.<init>(DataBaseORM.java:46)
	  at com.xxxx.video.common.data.DataBaseORM.getInstance(DataBaseORM.java:53)
	  locked <0x017095d5> (a java.lang.Class<com.xxxx.video.common.data.DataBaseORM>)
	  ```
- # 三、binder数据量过大
  collapsed:: true
	- ```text
	  07-21 04:43:21.573  1000  1488 12756 E Binder  : Unreasonably large binder reply buffer: on android.content.pm.BaseParceledListSlice$1@770c74f calling 1 size 388568 (data: 1, 32, 7274595)
	  07-21 04:43:21.573  1000  1488 12756 E Binder  : android.util.Log$TerribleFailure: Unreasonably large binder reply buffer: on android.content.pm.BaseParceledListSlice$1@770c74f calling 1 size 388568 (data: 1, 32, 7274595)
	  07-21 04:43:21.607  1000  1488  2951 E Binder  : Unreasonably large binder reply buffer: on android.content.pm.BaseParceledListSlice$1@770c74f calling 1 size 211848 (data: 1, 23, 7274595)
	  07-21 04:43:21.607  1000  1488  2951 E Binder  : android.util.Log$TerribleFailure: Unreasonably large binder reply buffer: on android.content.pm.BaseParceledListSlice$1@770c74f calling 1 size 211848 (data: 1, 23, 7274595)
	  07-21 04:43:21.662  1000  1488  6258 E Binder  : Unreasonably large binder reply buffer: on android.content.pm.BaseParceledListSlice$1@770c74f calling 1 size 259300 (data: 1, 33, 7274595)
	  ```
- # 四、binder 通信失败
  collapsed:: true
	- ```text
	  07-21 06:04:35.580 <6>[32837.690321] binder: 1698:2362 transaction failed 29189/-3, size 100-0 line 3042
	  07-21 06:04:35.594 <6>[32837.704042] binder: 1765:4071 transaction failed 29189/-3, size 76-0 line 3042
	  07-21 06:04:35.899 <6>[32838.009132] binder: 1765:4067 transaction failed 29189/-3, size 224-8 line 3042
	  07-21 06:04:36.018 <6>[32838.128903] binder: 1765:2397 transaction failed 29189/-22, size 348-0 line 2916
	  ```