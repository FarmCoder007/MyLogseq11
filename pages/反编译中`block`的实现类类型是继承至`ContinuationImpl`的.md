## 如果是挂起函数，做CPS变换后，将block转换成
	- ![Continuationimpl.gif](../assets/Continuationimpl_1693577169364_0.gif)
- ## 如果是Launch函数，
	- 将launch里的协程体block反编译为继承SuspendLambda 实现Function2，[[Continuation继承关系图]]得出结论