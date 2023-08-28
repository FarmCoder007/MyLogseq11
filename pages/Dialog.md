# 一、Dialog 的生命周期 六个
	- onCreate()，show()，onStart() ，cancel()，onDismiss()，Stop() 。
- # 二、生命周期调用场景
  collapsed:: true
	- 2-1 只new  [dialog](https://so.csdn.net/so/search?q=dialog&spm=1001.2101.3001.7020)（）  没调show方法    不走生命周期   只走自定义dialog的构造方法
	- 2-2 new  Dialog()    调研首次调show()
		- ![](https://img-blog.csdnimg.cn/2021052214352248.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1d2IxMjN4dXdi,size_16,color_FFFFFF,t_70)
	- 2-3 调dismiss()
		- ![](https://img-blog.csdnimg.cn/20210522144342158.png){:height 95, :width 719}
	- 2-4 非首次调show()   就不会走onCreat()
		- ![](https://img-blog.csdnimg.cn/20210522144436723.png)
	- 2-5 按back 或者 dialog 以外可消失：
		- ![](https://img-blog.csdnimg.cn/2021052214481365.png)
	- ## 总结：
		- 1onCreat() 只有首次调用show(） 才会走
		- 2 onStart()和 show() 在每次Dialog显示时都会依次执行。
		- 3stop() 和 onDismiss() 在每次Dialog消失的时候都会依次执行。
		- 4cancel() 是在点击BACK按钮或者Dialog外部时触发，依次执行stop()  onDismiss() cancel() 。
- # 三、注意应用场景
	- 1 当自定义dialog   再使用的时候 只 new 了dialog  而没有调用show()方法   此时不走任何生命周期，只走dialog的构造函数
	- ## 应用场景：延时show()，但是想先拿到view【解决没调show  findViewById 为null】
		- 可以将setContentView() 放在构造函数里，而不是放在onCreat()里 因为不会走
		- ```java
		  public LocationPermissionGuideDialog(Context context, int theme) {
		        super(context, theme);
		        mContext = context;
		        setContentView(R.layout.dialog_home_location);
		    }
		  ```
		- 这样获取到的view 就不会为null