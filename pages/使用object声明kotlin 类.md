- 这个类会自动创建一个单例对象   就可以直接调用   成员变量 方法 属性
- ## 调用时  会使用   类名,方法名   调用
	- ```java
	  // kotlin  使用 object声明类
	  object ToastUtils {
	      fun toast(context: Context, str: String?) {
	          Toast.makeText(context, str, Toast.LENGTH_LONG).show();
	      }
	  }
	   
	  // kotlin调用      直接类名.方法名
	  ToastUtils.toast(this,"Kotlin 调用")
	   
	  // java里调用   
	   ToastUtils.INSTANCE.toast(this,"java里调用 需要用INSTANCE实例");
	  ```
- ## 如果想java也能类名.调用 ，方法上需要加@JVMStatic注解
	- ```java
	  object ToastUtils {
	    	@JVMStatic
	      fun toast(context: Context, str: String?) {
	          Toast.makeText(context, str, Toast.LENGTH_LONG).show();
	      }
	  }
	  // java里调用   
	   ToastUtils.toast(this,"java里调用 需要用INSTANCE实例");
	  ```
- # 特点
	- object 修饰的类 没有主 和 次构造