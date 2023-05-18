- # 简介
  collapsed:: true
	- 协程是一种并发设计模式，您可以在 Android 平台上使用它来简化异步执行的代码。协程是在版本 1.3 中添加到 Kotlin 的，它基于来自其他语言的既定概念。
	- ## [特点](https://kotlinlang.org/docs/coroutines-basics.html#structured-concurrency)
		- 协程是我们在 Android 上进行异步编程的推荐解决方案。值得关注的特点包括：
		- 轻量：您可以在单个线程上运行多个协程，因为协程支持挂起，不会使正在运行协程的线程阻塞。挂起比阻塞节省内存，且支持多个并行操作。
		  内存泄漏更少：使用结构化并发机制在一个作用域内执行多项操作。
		  内置取消支持：取消操作会自动在运行中的整个协程层次结构内传播。
		  Jetpack 集成：许多 Jetpack 库都包含提供全面协程支持的扩展。某些库还提供自己的协程作用域，可供您用于结构化并发。
- # 原理解析
	- 本文着重于协程原理的剖析，不再进行协程使用部分的介绍，如不了解请参考其他使用文档。本文使用的kotlin版本为1.6.1，不同kotlin版本生成的字节码稍有不同，内部原理没有太大区别。
	- ## 源码分析
		- 为了详细展示出运行的原理，下面一段中包含大量代码，可以先阅读文字解说再看大致源码。
		- 第一步我们从最简单的代码开始，逐步剖析其中的原理，下面是我准备的第一段代码：
		  collapsed:: true
			- ```
			  class MainActivity : AppCompatActivity() {
			      @OptIn(DelicateCoroutinesApi::class)
			      override fun onCreate(savedInstanceState: Bundle?) {
			          super.onCreate(savedInstanceState)
			          setContentView(R.layout.activity_main)
			          GlobalScope.launch {
			              println("Coroutine start")
			          }
			      }
			  }
			  ```
		- 直接在Activity的onCreate方法中启动一个协程，打印一行文本。反编译class文件后得到如下两个类：
		  collapsed:: true
			- ```java
			  public final class MainActivity extends AppCompatActivity {
			      /* access modifiers changed from: protected */
			      @Override // androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, androidx.fragment.app.FragmentActivity
			      public void onCreate(Bundle savedInstanceState) {
			          super.onCreate(savedInstanceState);
			          setContentView(R.layout.activity_main);
			          Job unused = BuildersKt__Builders_commonKt.launch$default(GlobalScope.INSTANCE, null, null, new MainActivity$onCreate$1(null), 3, null);
			      }
			  }
			  ```
		- 代码
		  collapsed:: true
			- ```java
			  final class MainActivity$onCreate$1 extends SuspendLambda implements Function2<CoroutineScope, Continuation<? super Unit>, Object> {
			      int label;
			  
			      MainActivity$onCreate$1(Continuation<? super MainActivity$onCreate$1> continuation) {
			          super(2, continuation);
			      }
			  
			      @Override // kotlin.coroutines.jvm.internal.BaseContinuationImpl
			      public final Continuation<Unit> create(Object obj, Continuation<?> continuation) {
			          return new MainActivity$onCreate$1(continuation);
			      }
			  
			      public final Object invoke(CoroutineScope coroutineScope, Continuation<? super Unit> continuation) {
			          return ((MainActivity$onCreate$1) create(coroutineScope, continuation)).invokeSuspend(Unit.INSTANCE);
			      }
			  
			      @Override // kotlin.coroutines.jvm.internal.BaseContinuationImpl
			      public final Object invokeSuspend(Object obj) {
			          IntrinsicsKt.getCOROUTINE_SUSPENDED();
			          switch (this.label) {
			              case 0:
			                  ResultKt.throwOnFailure(obj);
			                  System.out.println((Object) "Coroutine start");
			                  return Unit.INSTANCE;
			              default:
			                  throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
			          }
			      }
			  }
			  ```
		- 从上面反编译类可以看出，协程代码块会被编译成继承了SuspendLambda的代码类，并实现了Function2接口，Function2是高阶函数生成的类，那么这个类的invoke方法就是入口方法。看到这里我们可以暂时把这个类放一放，看看上面的MainActivity的反编译代码。
		- 协程的启动被编译成了这行代码
		  collapsed:: true
			- ```
			      Job unused = BuildersKt__Builders_commonKt.launch$default(GlobalScope.INSTANCE, null, null, new MainActivity$onCreate$1(null), 3, null);`
			  ```
		- 看一下BuildersKt__Builders_commonKt.launch$default方法
		  collapsed:: true
			- ```
			    public static /* synthetic */ Job launch$default(CoroutineScope coroutineScope, CoroutineContext coroutineContext, CoroutineStart coroutineStart, Function2 function2, int i, Object obj) {
			          if ((i & 1) != 0) {
			              coroutineContext = EmptyCoroutineContext.INSTANCE;
			          }
			          if ((i & 2) != 0) {
			              coroutineStart = CoroutineStart.DEFAULT;
			          }
			          return BuildersKt.launch(coroutineScope, coroutineContext, coroutineStart, function2);
			      }
			  ```
		- 先不关注细枝末节，继续看BuildersKt.launch方法
		  collapsed:: true
			- ```
			      public static final Job launch(CoroutineScope $this$launch, CoroutineContext context, CoroutineStart start, Function2<? super CoroutineScope, ? super Continuation<? super Unit>, ? extends Object> function2) {
			          return BuildersKt__Builders_commonKt.launch($this$launch, context, start, function2);
			      }
			  ```
		- 再回到BuildersKt__Builders_commonKt.launch方法
		  collapsed:: true
			- ```
			      public static final Job launch(CoroutineScope $this$launch, CoroutineContext context, CoroutineStart start, Function2<? super CoroutineScope, ? super Continuation<? super Unit>, ? extends Object> function2) {
			          LazyStandaloneCoroutine coroutine;
			          CoroutineContext newContext = CoroutineContextKt.newCoroutineContext($this$launch, context);
			          if (start.isLazy()) {
			              coroutine = new LazyStandaloneCoroutine(newContext, function2);
			          } else {
			              coroutine = new StandaloneCoroutine(newContext, true);
			          }
			          coroutine.start(start, coroutine, function2);
			          return coroutine;
			      }
			  ```
		- 继续追踪coroutine.start方法，这里有一个lazy的判断，简单看一下可知LazyStandaloneCoroutine是继承自StandaloneCoroutine，他们的start的实现都在父类AbstractCoroutine中，具体实现如下：
		  collapsed:: true
			- ```
			    public final <R> void start(CoroutineStart start, R r, Function2<? super R, ? super Continuation<? super T>, ? extends Object> function2) {
			          start.invoke(function2, r, this);
			      }
			  ```
		- 继续往下追踪，CoroutineStart.invoke(function2, r, this)方法由于反编译后的代码由于有枚举类阅读起来比较吃力，我们可以直接看kotlin源码：
		- 在分析kotlin协程源码时要结合反编译和kotlin源码，由于kotlin编译器会在编译时生成较多代码因此优先读反编译代码，当发现反编译代码阅读比较吃力时再切换到kotlin源码方式。
		  collapsed:: true
			- ```
			      public operator fun <R, T> invoke(block: suspend R.() -> T, receiver: R, completion: Continuation<T>): Unit =
			          when (this) {
			              DEFAULT -> block.startCoroutineCancellable(receiver, completion)
			              ATOMIC -> block.startCoroutine(receiver, completion)
			              UNDISPATCHED -> block.startCoroutineUndispatched(receiver, completion)
			              LAZY -> Unit // will start lazily
			          }
			  
			  ```
		- 看到这里我们就不能继续往下读了，因为我们不知道CoroutineStart这个类是什么枚举类，那就回到CoroutineStart这个类创建的地方。回到MainActivity类，发现BuildersKt__Builders_commonKt.launch$default方法中传的CoroutineStart参数是null，再往下走BuildersKt__Builders_commonKt.launch方法中有一段代码如下：
		  collapsed:: true
			- ```
			  if ((i & 2) != 0) {
			     coroutineStart = CoroutineStart.DEFAULT;
			  }
			  ```
		- 通过上文我们可知这个i=3，因此coroutineStart就是CoroutineStart.DEFAULT。那我们就继续往下看：
			- ```
			  ```
-