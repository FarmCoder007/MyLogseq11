## onStartCommand()返回值的区别，服务被系统杀死后，怎么操作#Card
id:: 64e9612d-2524-4514-8591-95d87f1c6f29
	- ## 方法调用时机：每次服务启动的时候调用
	- ## 1、START_NOT_STICKY（常量值：2）
		- - 默认情况下，系统不会重新创建服务。除非有将要传递来的Intent时，系统重新创建服务，并调用`onStartCommand()`，传入此Intent。
		- ==车祸后再也没有苏醒==
		- - 这是最安全的选项，可以避免在不必要的时候运行服务。
	- ## 2、START_STICKY（常量值：1）
		- 系统重新创建服务，并调用`onStartCommand()`，默认是传入空Intent。除非存在将要传递来的Intent，那么就传递这些Intent。
		- ==特点：车祸后自己苏醒，但是失忆==
		- 适合：播放器一类的服务。独立运行，但无需执行命令，只等待任务。
	- ## 3、START_REDELIVER_INTENT（常量值：3）
		- - 系统重新创建服务，并调用`onStartCommand()`，传入上次传递给服务执行过的Intent。
		- ==车祸后自己苏醒，依然保持记忆==。
		- - 适合像下载一样的服务。立即恢复，积极执行。