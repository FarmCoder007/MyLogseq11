- # 前言
	- 在前面文章中，我们重点分析了挂起函数的原理，包括知识点有：挂起函数内部其实就是`CPS`加状态机的模型，`Continuation`类似于`Callback`，即可以用于实现挂起函数向挂起函数外传递数据，也可以使用匿名内部类方式接收挂起函数返回值，最后就是创建挂起函数的最底层函数`suspendCoroutineUninterceptedOrReturn`方法其实就是为了实现状态机逻辑，同时消除`suspend`关键字。
- #  协程启动的基础API
	- 什么是基础API呢？其实我们前面所说的启动协程的方法，比如`launch`、`async`都是属于上层或者中间层API，它们都是调用了基础API。
	- 既然这么重要，我们就来看看：
		- ```kotlin
		  //创建协程
		  public fun <R, T> (suspend R.() -> T).createCoroutine(
		      receiver: R,
		      completion: Continuation<T>
		  ): Continuation<Unit> =
		      SafeContinuation(createCoroutineUnintercepted(receiver, completion).intercepted(), COROUTINE_SUSPENDED)
		  
		  //启动协程
		  public fun <T> (suspend () -> T).startCoroutine(
		      completion: Continuation<T>
		  ) {
		      createCoroutineUnintercepted(completion).intercepted().resume(Unit)
		  }
		  ```
	- 这里可以发现`createCoroutine{}`和`startCoroutine{}`都是扩展函数，而且扩展的接收者类型是`(suspend () -> T)`。
	- 或许我们经常给一些常用的类型添加扩展函数，但是几乎没有干过给函数类型添加扩展函数。既然Kotlin中，函数作为一等公民，我们给它添加扩展函数也是可以的。
	- 那我们如何调用上面扩展函数呢？测试代码如下：
	  collapsed:: true
		- ```kotlin
		  fun main(){
		      Thread.sleep(2000L)
		      testStartCoroutine()
		  }
		  
		  /**
		   * 这里的block类型是"suspend () -> String"
		   *
		   * 这里我们秉承 单方法接口 <--> 高阶函数 <--> lambda这种关系
		   * */
		  val block = suspend {
		      println("Hello")
		      delay(1000L)
		      println("Kotlin")
		      "Result"
		  }
		  
		  /**
		   * 这里调用了[startCoroutine]扩展函数，这个扩展函数是 suspend () -> T 的
		   * 扩展函数。
		   *
		   * [Continuation]有2个作用，一个是实现挂起函数时用来向外传递数据；一个是以匿名
		   * 内部类的方式来接收一个挂起函数的值。
		   * */
		  private fun testStartCoroutine(){
		      val continuation = object : Continuation<String>{
		          override val context: CoroutineContext
		              get() = EmptyCoroutineContext
		  
		          override fun resumeWith(result: Result<String>) {
		              println("Result is ${result.getOrNull()}")
		          }
		      }
		  
		      block.startCoroutine(continuation)
		  }
		  ```
	- -
	- 这里定义了变量名为`block`的`lambda`表达式，它的类型是`suspend () -> String`，因为`lambda`表达式最后一行是该`lambda`的返回值；同时在Kotlin中，高阶函数、单接口方法、lambda可以看成是一样的。
	- -
	- 然后定义了一个`continuation`变量，根据前一篇文章我们知道`Continuation`有2个作用：一种是在实现挂起函数的时候，用于传递挂起函数的执行结果；另一种是在调用挂起函数的时候，以匿名内部类的方式，接收挂起函数的执行结果。而上面代码的作用就是第二种，用来接收`block`的执行结果。
	- 这里的这种使用方法，就感觉像是给一个挂起函数设置了`Continuation`参数一样，根据前面`CPS`原理，我们知道每个挂起函数都需要一个`Continuation`参数追加到参数列表后，那这里真是这样吗？
	- 我们可以通过分析源码来解读一下。
- #  `startCoroutine{}` 原理解析
	- 这里我们直接把上面代码进行反编译，可以得到如下Java代码：
	  collapsed:: true
		- ```kotlin
		  public final class KtCreateCoroutineKt {
		     @NotNull
		     private static final Function1 block;
		  
		     //注释1 主函数调用
		     public static final void main() {
		        Thread.sleep(2000L);
		        testStartCoroutine();
		     }
		  
		     // $FF: synthetic method
		     public static void main(String[] var0) {
		        main();
		     }
		  
		     @NotNull
		     public static final Function1 getBlock() {
		        return block;
		     }
		  
		     //注释2 这里创建了一个Continuation对象，但是类型无法解析
		     //这是因为它是一个匿名内部类 
		     private static final void testStartCoroutine() {
		        <undefinedtype> continuation = new Continuation() {
		           @NotNull
		           public CoroutineContext getContext() {
		              return (CoroutineContext)EmptyCoroutineContext.INSTANCE;
		           }
		  
		           public void resumeWith(@NotNull Object result) {
		              String var2 = "Result is " + (String)(Result.isFailure-impl(result) ? null : result);
		              System.out.println(var2);
		           }
		        };
		        //注释3 扩展函数变成了Java静态方法调用，参数为block和continuation
		        ContinuationKt.startCoroutine(block, (Continuation)continuation);
		     }
		  
		     static {
		        //注释4，lambda原本是一个无参高阶函数，这里默认会添加一个Continuation
		        //同样的，这里是匿名内部类的原因，无法具体解析出var0的类型
		        Function1 var0 = (Function1)(new Function1((Continuation)null) {
		           int label;
		  
		           //CPS后的状态机逻辑，当调用continuaiton的resume方法，会回调如此。
		           //这里的0分支中，调用delay后，会挂起，进入delay方法中，并且参数this也就是var0自己
		           //调用完delay后，进入1分支，同时打印Kotlin，返回Result字段
		           @Nullable
		           public final Object invokeSuspend(@NotNull Object $result) {
		              Object var3 = IntrinsicsKt.getCOROUTINE_SUSPENDED();
		              String var2;
		              switch(this.label) {
		              case 0:
		                 ResultKt.throwOnFailure($result);
		                 var2 = "Hello";
		                 System.out.println(var2);
		                 this.label = 1;
		                 if (DelayKt.delay(1000L, this) == var3) {
		                    return var3;
		                 }
		                 break;
		              case 1:
		                 ResultKt.throwOnFailure($result);
		                 break;
		              default:
		                 throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
		              }
		  
		              var2 = "Kotlin";
		              System.out.println(var2);
		              return "Result";
		           }
		  
		           //注释5 根据一个Continuation对象，创建一个新的Continuation对象，其实这个类型就是
		           //状态机中的Continuation类型，即block实现类的类型
		           @NotNull
		           public final Continuation create(@NotNull Continuation completion) {
		              Intrinsics.checkNotNullParameter(completion, "completion");
		              Function1 var2 = new <anonymous constructor>(completion);
		              return var2;
		           }
		  
		           public final Object invoke(Object var1) {
		              return ((<undefinedtype>)this.create((Continuation)var1)).invokeSuspend(Unit.INSTANCE);
		           }
		        });
		        block = var0;
		     }
		  }
		  ```
	- 这里反编译的代码，如果看过文章[# 协程(15) | 挂起函数原理解析](https://juejin.cn/post/7094235540584661029) 中的`CPS`后的状态机原理，就不难理解，代码中关键地方，都进行了注释标注。
	- 我们还是来简单说明一下：
	- -
	- 注释1、2、3是`testStartCoroutine()`方法的调用，这里使用匿名内部类的方式，把`Continuation`对象传递给`startCoroutine`函数。
	- -
	- 注释4就是典型的状态机逻辑，就是把原来`suspend () -> String`类型的`block`转换为`var0`，在这其中注释4的逻辑就是`CPS`后的状态机逻辑，里面有2个分支，因为在这里面我们调用了`delay`挂起函数。
	- -
	- 不同于普通的匿名内部类实现，在这里多了注释5的方法，这说明`var0`所实现的接口中有`create()`方法，在该方法中，会根据一个`Continuation`参数创建`var0`。
	- 这个var0其实就是`block`这一段`lambda`在经过编译器处理后的对象，其类型我们目前只知道是`Continuation`的子类。
	- 我们接着来看一下`startCoroutine{}`的源码实现：
		- ```kotlin
		  public fun <T> (suspend () -> T).startCoroutine(
		      completion: Continuation<T>
		  ) {
		      createCoroutineUnintercepted(completion).intercepted().resume(Unit)
		  }。
		  ```
	- 这里调用了`createCoroutineUnintercepted()`方法：
		- ```kotlin
		  public expect fun <T> (suspend () -> T).createCoroutineUnintercepted(
		      completion: Continuation<T>
		  ): Continuation<Unit>
		  ```
	- 会发现这里是用`expect`修饰的，即是一种声明，我们需要到协程源代码的JVM实现部分中找到对应的实现：
		- ```kotlin
		  public actual fun <T> (suspend () -> T).createCoroutineUnintercepted(
		      completion: Continuation<T>
		  ): Continuation<Unit> {
		      val probeCompletion = probeCoroutineCreated(completion)
		      //注释1
		      return if (this is BaseContinuationImpl)
		          create(probeCompletion)
		      else
		          createCoroutineFromSuspendFunction(probeCompletion) {
		              (this as Function1<Continuation<T>, Any?>).invoke(it)
		          }
		  }
		  ```
	- 可以发现这也是`(suspend () -> T)`的扩展函数，所以`this`其实就是前面代码中的`block`。
	- 这里需要注意了，前面我们说的反编译中`block`的实现类类型是继承至`ContinuationImpl`的，这个十分重要，因为反编译代码无法完整显示出，所以注释2的第一个`if`就能返回`ture`，而这里就是调用`create(probeCompletion)`函数。
	- 而这个`create()`方法就是前面反编译中`block`实现类的`create()`方法：
		- ```kotlin
		  @NotNull
		  public final Continuation create(@NotNull Continuation completion) {
		     Intrinsics.checkNotNullParameter(completion, "completion");
		     Function1 var2 = new <anonymous constructor>(completion);
		     return var2;
		  }
		  ```
	- 在这个`create`方法中，会把我们传入的`continuation`对象进行包裹，再次返回一个`Continuation`对象，根据前面文章挂起函数原理可知，这个其实就相当于第一次进入状态机，我们新建一个`Continuation`对象，而这个对象类型就是`var0`的实现类类型。
	- 注意了，这里返回值是`Continuation`类型对象，即调用完`create()`方法，其实就对应着协程被创建了，和挂起函数一样，类型是`Continuation`类型。
	- 所以这里就好办了，根据前面的知识，这时调用`resume`，[[#red]]==**便会触发协程体的状态机入口**==，所以
		- ```kotlin
		  public fun <T> (suspend () -> T).startCoroutine(
		      completion: Continuation<T>
		  ) {
		      createCoroutineUnintercepted(completion).intercepted().resume(Unit)
		  }
		  ```
	- 这里的最后调用就是`resume(Unit)`，调用完`resume`就会调用`continuation`的`invokeSuspend`方法，从而开启协程的执行。
	- 注意上面在`resume()`方法调用之前，还调用了`intercepted()`方法，我们简单看一下：
		- ```kotlin
		  public expect fun <T> Continuation<T>.intercepted(): Continuation<T>
		  ```
	- 这个方法在`Continuation.kt`类中，是基础元素，同时也是用`expect`修饰的，所以我们要去Kotlin源码中找到JVM平台的实现：
		- ```kotlin
		  public actual fun <T> Continuation<T>.intercepted(): Continuation<T> =
		      (this as? ContinuationImpl)?.intercepted() ?: this
		  ```
	- 这里逻辑非常简单，就是`将Continuation`强转为`ContinuationImpl`，然后调用它的`intercpeted()`方法，而前面我们说过`block`实现类就是这个类的子类，所以强转一定能成功，而这个方法如下
		- ```kotlin
		  internal abstract class ContinuationImpl(
		      completion: Continuation<Any?>?,
		      private val _context: CoroutineContext?
		  ) : BaseContinuationImpl(completion) {
		  
		      @Transient
		      private var intercepted: Continuation<Any?>? = null
		  
		      public fun intercepted(): Continuation<Any?> =
		          intercepted
		              ?: (context[ContinuationInterceptor]?.interceptContinuation(this) ?: this)
		                  .also { intercepted = it }
		  }
		  ```
	- 这里的逻辑其实就是通过`ContinuationInterceptor`类来对`Continuation`进行拦截和处理，而这里的处理其实就是将协程派发到线程上，这部分知识点等我们说`Dispatchers`时再细说。
	- 所以到这里我们就大致说明白了底层启动协程API的原理，其中`block`就是一个协程，它的类型必须是`suspend`类型的，然后本质就是一个内部类实例，父类是`Function1`和`ContinuationImpl`，创建完协程就是返回一个内部类实例，即状态机。
	- 然后调用`resume(Unit)`方法来触发状态机的`invokeSuspend`方法，从而开始其状态机逻辑。
	-
- # `createCoroutine{}` 原理分析
  collapsed:: true
	- 和`startCoroutine{}`对应的还有一个创建协程的基础API，方法如下：
		- ```kotlin
		  public fun <T> (suspend () -> T).createCoroutine(
		      completion: Continuation<T>
		  ): Continuation<Unit> =
		      SafeContinuation(createCoroutineUnintercepted(completion).intercepted(), COROUTINE_SUSPENDED)
		  ```
	- 从这里我们发现，它是一样调用了`createCoroutineUnintercrepted`方法，但是没有调用`resume(Unit)`，即没有进入状态机。
	- 所以上面测试代码，和下面写法是一样的：
		- ```kotlin
		  private fun testCreateCoroutine(){
		      val continuation = object : Continuation<String>{
		          override val context: CoroutineContext
		              get() = EmptyCoroutineContext
		  
		          override fun resumeWith(result: Result<String>) {
		              println("Result is ${result.getOrNull()}")
		          }
		      }
		      //这里手动调用resume(Unit)方法
		      val c = block.createCoroutine(continuation)
		      c.resume(Unit)
		  }
		  ```
- # 总结
	- 本篇文章我们见识到了创建协程的底层API，即：`startCoroutine{}`和`createCoroutine{}`，这个方法是`suspend () -> T`挂起函数的扩展函数，根据挂起函数`CPS`后的原理，它需要传入一个`Continuation`，而该方式下，挂起函数的实现类，会继承`ContinuationImpl`类，该类中有`create()`方法，从而产生一个`Continuation`类型的状态机对象。
	- 最后调用`resume`方法来开启状态机。
	- 学习完本篇文章，我们就知道，其实协程就是对挂起函数的进一步处理，下篇文章我们就来仔细看看启动协程的`launch`函数的原理。