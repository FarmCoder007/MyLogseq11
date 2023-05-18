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
-