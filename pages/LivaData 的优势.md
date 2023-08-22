title:: LivaData 的优势

- ## 1、LiveData 能确保 UI 和数据状态相符
	- LiveData 遵循观察者模式。当底层数据发生变化时，LiveData 会通知 Observer 对象来更新界面。
- ## 2、不会发生内存泄漏
	- 观察者和 Lifecycle 对象绑定，能在销毁时自动解除注册。
- ## 3、不会给已经停止的 Activity 发送事件
	- 如果观察者处于非活跃状态，LiveData 不会再发送任何事件给这些 Observer 对象。
- ## 4、不再需要手动处理生命周期
	- UI 组件仅仅需要对相关数据进行观察，LiveData 自动处理生命周期状态改变后，需要处理的代码。
- ## 5、数据始终保持最新状态
	- 一个非活跃的组件进入到活跃状态后，会立即获取到最新的数据，不用担心数据问题。
- ## 6、LiveData 在横竖屏切换等 Configuration 改变时，也能保证获取到最新数据，这个是viiewModel的作用
	- 例如 Acitivty、Fragment 因为屏幕选装导致重建, 能立即接收到最新的数据。
- ## 7、LiveData 能资源共享
	- 如果将 LiveData 对象扩展，用单例模式将系统服务进行包裹。这些服务就可以在 APP 中共享。