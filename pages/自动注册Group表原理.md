## 背景
	- 传统方式在初始化时遍历Dex文件 查找Root类，项目比较大时，会非常耗时
- ## Gradle插件自动注册Group表，将收集过程提前到编译期
	- 1、自定义Gradle插件，注册Transform（执行时机在class到dex中间）拿到所有编译后的class文件
	- 2、利用ASM框架的 ClassVisitor 扫描 calss文件，收集所有的ARouter$Root$xxx 和 记录 LogisticsCenter 所在jar包的位置
	- 3、利用ASM框架的 MethodVisitor 在 LogisticsCenter 中预留方法中loadRouteMap中插入生成注册root的代码
	-
	- 注意：
		- 4 修改字节码并不是在修改原class文件，而是把class拷贝了一份到输出路径也就是下个Transform的输入数据，改完后从classWriter得到修改后的byte流，然后写入到输出路径,流向下一个Trasform达到在编译期操作字节码的目的。