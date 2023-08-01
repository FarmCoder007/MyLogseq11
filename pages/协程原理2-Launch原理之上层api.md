- # 前言
	- 在上一篇文章中，我们介绍了开启协程：`startCoroutine{}`和创建协程：`startCoroutine{}`这2个基础API，虽然这2个方法可以创建和开启协程，但是它们并没有前面我们所说的协程相关的概念，比如`Job`、`CoroutineScope`等，所以他们只能算是非常基础的API。
	- 这篇文章，我们来看看启动协程的上层`launch{}`函数的原理，看看是如何在调用基础API的同时，加入我们一些熟知的协程概念。
- #  `launch{}` 函数分析
	- 话不多说，我们直接看个例子：
		- ```kotlin
		  fun main() {
		      testLaunch()
		      Thread.sleep(2000L)
		  }
		  
		  private fun testLaunch() {
		      val scope = CoroutineScope(Job())
		      scope.launch {
		          println("Hello!")
		          delay(1000L)
		          println("World!")
		      }
		  }
		  ```
	- 这里我们创建了一个`CoroutineScope`，然后使用其扩展函数`launch{}`来开启协程，在这里`launch{}`中的`lambda`表达式类型是`suspend Coroutine.() -> Unit`，所以上面代码可以改写成下面这样：
		- ```kotlin
		  private fun testCoroutine(){
		      val scope = CoroutineScope(Job())
		      val b: suspend CoroutineScope.() -> Unit = {
		          println("Hello")
		          delay(1000)
		          println("Kotlin")
		      }
		      scope.launch(block = b)
		  }
		  ```
	- 看到这里，是不是有一种上篇文章中的影子，上一篇文章中，我们使用的例子是这样的：
		- ```kotlin
		  val block = suspend {
		      println("Hello")
		      delay(1000L)
		      println("Kotlin")
		      "Result"
		  }
		  
		  val c = block.createCoroutine(continuation)
		  c.resume(Unit)
		  ```
	- 我们直接把上面代码反编译为Java，看看`CPS`后的结果，是什么效果，代码如下：
	  collapsed:: true
		- ```kotlin
		  //反编译后的代码
		  private static final void testCoroutine() {
		  //创建CoroutineScope
		     CoroutineScope scope = CoroutineScopeKt.CoroutineScope((CoroutineContext)JobKt.Job$default((Job)null, 1, (Object)null));
		     //block对应的状态机逻辑代码
		     Function2 b = (Function2)(new Function2((Continuation)null) {
		        int label;
		  
		        //状态机逻辑，在这里由于原来block中调用了delay挂起函数，所以这里状态机有2个分支
		        //在0分支中，就是执行了delay挂起函数，打印了Hello，在1分支中，打印Kotlin，该
		        //方法会被调用2次
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
		           return Unit.INSTANCE;
		        }
		  
		        //创建状态机实例，参数为Continuation实例，返回类型也是Continuation对象
		        @NotNull
		        public final Continuation create(@Nullable Object value, @NotNull Continuation completion) {
		           Intrinsics.checkNotNullParameter(completion, "completion");
		           Function2 var3 = new <anonymous constructor>(completion);
		           return var3;
		        }
		  
		        public final Object invoke(Object var1, Object var2) {
		           return ((<undefinedtype>)this.create(var1, (Continuation)var2)).invokeSuspend(Unit.INSTANCE);
		        }
		     });
		     //调用launch函数
		     BuildersKt.launch$default(scope, (CoroutineContext)null, (CoroutineStart)null, b, 3, (Object)null);
		  }
		  ```
	- 这里可以发现反编译后的Java代码非常符合我们的预期，这里`block`是挂起函数，同时也是协程所执行的代码块。
	- 其中状态机类型自己就是`Continuation`的子类，这里唯一和上一篇文章不同的是`create()`方法的参数需要一个`Continuation`实例来接收挂起函数的执行结果，那`launch{}`中是否有对应逻辑呢？
	-
- # `launch{}` 原理分析
	- 下面是`launch{}`的源码
		- ```kotlin
		  public fun CoroutineScope.launch(
		      context: CoroutineContext = EmptyCoroutineContext,
		      start: CoroutineStart = CoroutineStart.DEFAULT,
		      block: suspend CoroutineScope.() -> Unit
		  ): Job {
		      val newContext = newCoroutineContext(context)
		      val coroutine = if (start.isLazy)
		          LazyStandaloneCoroutine(newContext, block) else
		          StandaloneCoroutine(newContext, active = true)
		      coroutine.start(start, coroutine, block)
		      return coroutine
		  }
		  ```
	- 分析如下：
	- 首先是`block`的类型，它是挂起函数类型，经过我们之前的经验，想启动挂起函数，需要传递一个`Continuation`对象。
	- 默认的`start`是`DEFAULT`，所以这里的`coroutine`是`StandaloneCoroutine`实例。`Standalone`翻译为中文是独立运行的。
	- 这里的`newCoroutineContext`是`CoroutineScope`的扩展函数，既然这个协程是该协程范围内启动的，那就应该拿到该协程范围的上下文信息。
	- 最后调用`start`方法，该方法如下：
		- ```kotlin
		  public fun <R> start(start: CoroutineStart, receiver: R, block: suspend R.() -> T) {
		      start(block, receiver, this)
		  }
		  ```
	- 这里使用了`invoke`约定的简写，调用的其实就是`DEFAULT`这个类的`invoke`方法：
		- ```kotlin
		  public operator fun <T> invoke(block: suspend () -> T, completion: Continuation<T>): Unit =
		      when (this) {
		          DEFAULT -> block.startCoroutineCancellable(completion)
		          ATOMIC -> block.startCoroutine(completion)
		          UNDISPATCHED -> block.startCoroutineUndispatched(completion)
		          LAZY -> Unit // will start lazily
		      }
		  ```
	- 这里注意`DEFAULT`分支，调用了`block`的扩展方法：
		- ```kotlin
		  public fun <T> (suspend () -> T).startCoroutineCancellable(completion: Continuation<T>): Unit = runSafely(completion) {
		      createCoroutineUnintercepted(completion).intercepted().resumeCancellableWith(Result.success(Unit))
		  }
		  ```
	- 这里的`createCoroutineUnintercepted`方法就是上一篇文章我们所说的方法，它会调用上面反编译后Java代码中的`create()`方法，从而创建状态机实例，然后调用其`resume`方法，便开始状态机运行，到这里，`launch{}`就创建且启动了一个协程。
- # `AbstractCoroutine` 类简析
	- 看到这里我们知道协程代码块已经运行了，但是这个`Continuation`对象是什么时候创建的呢？
	- 核心代码就是：
		- ```kotlin
		  val coroutine = if (start.isLazy)
		      LazyStandaloneCoroutine(newContext, block) else
		      StandaloneCoroutine(newContext, active = true)
		  ```
	- 从上面`start`方法分析，我们可以知道这个`coroutine`类型其实就是`Continuation`类型，我们来看一下其实现：
		- ```kotlin
		  private open class StandaloneCoroutine(
		      parentContext: CoroutineContext,
		      active: Boolean
		  ) : AbstractCoroutine<Unit>(parentContext, initParentJob = true, active = active) {
		      override fun handleJobException(exception: Throwable): Boolean {
		          handleCoroutineException(context, exception)
		          return true
		      }
		  }
		  ```
	- 该类继承至`AbstractCoroutine`类：
		- ```kotlin
		  public abstract class AbstractCoroutine<in T>(
		      parentContext: CoroutineContext,
		      initParentJob: Boolean,
		      active: Boolean
		  ) : JobSupport(active), Job, Continuation<T>, CoroutineScope
		  ```
	- 这个类是协程构建器中实现协程的抽象基类，可以发现它实现了4个接口，其中：
		- `Job`接口就是我们之前说的协程句柄，通过句柄我们可以查看协程的状态，以及控制协程，同时`launch{}`方法返回的也是`Job`类型，在源码中我们直接返回`coroutine`变量即可。
		- `Continuation`接口是挂起函数所必须的，这里的`Continuation`作用就是接收挂起函数的返回值，我们可以看一下它的接口实现：
		- ```Kotlin
		  
		  //Continuation接口中的属性
		  public final override val context: CoroutineContext = parentContext + this
		  ```
		- 这里的`parentContext`就是协程范围或者父协程的上下文，
	- ```kotlin
	  //Continuation接口中的resumeWith方法实现
	  public final override fun resumeWith(result: Result<T>) {
	      val state = makeCompletingOnce(result.toState())
	      if (state === COMPLETING_WAITING_CHILDREN) return
	      afterResume(state)
	  }
	  。
	  ```
	- 这里会根据挂起函数的执行结果，调用`makeCompletingOnce`来改变`Job`的状态。
		- - `JobSupport`是一个实现`Job`接口的类，我们可以使用该类的实现方法，相当于一个默认实现。
		- - `CoroutineScope`接口表示协程范围，其实现是：
			- ```kotlin
			  public override val coroutineContext: CoroutineContext get() = context
			  ```
	- 从这里来看，`AbstractCoroutine`真是一个集大成者，这样也就让`launch{}`源码中的`coroutine`有了更多作用，这也就解释了`Continuation`的创建时机，至于该对象的其他作用，其实我们从继承和实现的接口就能看出，比如`Job`可以控制协程，`CoroutineScope`来控制协程结构化等，这些知识我们后面再分析。
- # 总结
	- 本篇文章以上一篇文章为基石，首先我们知道`launch{}`函数的`block`参数其实就是挂起函数，通过给定`Continuation`对象，以及调用其`resume`方法就可以运行挂起函数。
	- 其次是关键的`AbstractCoroutine`类的对象，它实现了多种接口，其中就包括`Job`和`Continuation`，也通过该对象实例，让一段挂起函数多了很多协程概念。