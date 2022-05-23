- 调用含有可变参数类型的方法时，需要传*
- 例子：
	- ```
	  fun main(args: Array<String>) {
	      invokMsg("AAA", "BBB", "CCC", "DDD")
	  }
	  
	  fun invokMsg(vararg value: Any?){
	      printlnMsg(value)
	      printlnMsg(*value)
	  }
	  
	  fun printlnMsg(vararg msg: Any?){
	      println(msg.joinToString(" "))
	  }
	  ```
	- ```
	  //输出了[Ljava.lang.Object;@a09ee92（类的对象名称）
	  [Ljava.lang.Object;@a09ee92
	  // 带*输出了值
	  AAA BBB CCC DDD
	  
	  Process finished with exit code 0
	  ```
- 查询原因
	- 通过查看kotlin转换的java源码来看看问题到底出在哪儿
	  Android Studio -> Tools -> Kotlin -> Show Kotlin Bytecode
	- ```
	  public final class VarargDemoKt {
	     public static final void main(@NotNull String[] args) {
	        Intrinsics.checkParameterIsNotNull(args, "args");
	        invokMsg("AAA", "BBB", "CCC", "DDD");
	     }
	  
	     public static final void invokMsg(@NotNull Object... value) {
	        Intrinsics.checkParameterIsNotNull(value, "value");
	        printlnMsg((Object)value);
	        printlnMsg(Arrays.copyOf(value, value.length));
	     }
	  
	     public static final void printlnMsg(@NotNull Object... msg) {
	        Intrinsics.checkParameterIsNotNull(msg, "msg");
	        String var1 = ArraysKt.joinToString$default(msg, (CharSequence)" ", (CharSequence)null, (CharSequence)null, 0, (CharSequence)null, (Function1)null, 62, (Object)null);
	        boolean var2 = false;
	        System.out.println(var1);
	     }
	  }
	  
	  ```
	- 当我们 添加 * 时，在java代码中会重新复制一个新的数组 Arrays.copyOf(value, value.length)，而不添加 *时，直接是将我们传入的数据转换成一个Object类型，这也就能解释为什么我们的日志输出一个是类对象的长度，一个是我们想要的输出值了