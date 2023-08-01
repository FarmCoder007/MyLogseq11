- ## 反编译前
	- ```kotlin
	      fun enter(){
	          GlobalScope.launch {
	              val data = goodTest()
	              print(data)
	          }
	          Thread.sleep(3000)
	  
	      }
	  ```
- func2 即为block CPS变换后，转换为
	- 实现 SuspendLamada 和Function2的 类
- ## 反编译变换前
  collapsed:: true
	- ```kotlin
	     public final void enter() {
	        BuildersKt.launch$default((CoroutineScope)GlobalScope.INSTANCE, (CoroutineContext)null, (CoroutineStart)null, (Function2)(new Function2((Continuation)null) {
	           int label;
	  
	           @Nullable
	           public final Object invokeSuspend(@NotNull Object $result) {
	              Object var3 = IntrinsicsKt.getCOROUTINE_SUSPENDED();
	              Object var10000;
	              switch (this.label) {
	                 case 0:
	                    ResultKt.throwOnFailure($result);
	                    Test2 var4 = Test2.this;
	                    this.label = 1;
	                    var10000 = var4.goodTest(this);
	                    if (var10000 == var3) {
	                       return var3;
	                    }
	                    break;
	                 case 1:
	                    ResultKt.throwOnFailure($result);
	                    var10000 = $result;
	                    break;
	                 default:
	                    throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
	              }
	  
	              String data = (String)var10000;
	              System.out.print(data);
	              return Unit.INSTANCE;
	           }
	  
	           @NotNull
	           public final Continuation create(@Nullable Object value, @NotNull Continuation completion) {
	              Intrinsics.checkNotNullParameter(completion, "completion");
	              Function2 var3 = new <anonymous constructor>(completion);
	              return var3;
	           }
	  
	           public final Object invoke(Object var1, Object var2) {
	              return ((<undefinedtype>)this.create(var1, (Continuation)var2)).invokeSuspend(Unit.INSTANCE);
	           }
	        }), 3, (Object)null);
	        Thread.sleep(3000L);
	     }
	  ```
- block对应的状态机逻辑代码，即放入了中,将匿名对象提出来 等价于下边代码
- ## 反编译整理后
  collapsed:: true
	- ```kotlin
	     public final void enter() {
	       //block对应的状态机逻辑代码	
	        Function2  func2 = new Function2((Continuation)null) {
	           int label;
	  
	           @Nullable
	           public final Object invokeSuspend(@NotNull Object $result) {
	              Object var3 = IntrinsicsKt.getCOROUTINE_SUSPENDED();
	              Object var10000;
	              switch (this.label) {
	                 case 0:
	                    ResultKt.throwOnFailure($result);
	                    Test2 var4 = Test2.this;
	                    this.label = 1;
	                    var10000 = var4.goodTest(this);
	                    if (var10000 == var3) {
	                       return var3;
	                    }
	                    break;
	                 case 1:
	                    ResultKt.throwOnFailure($result);
	                    var10000 = $result;
	                    break;
	                 default:
	                    throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
	              }
	  
	              String data = (String)var10000;
	              System.out.print(data);
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
	        }
	        // 启动launch方法
	        BuildersKt.launch$default((CoroutineScope)GlobalScope.INSTANCE, (CoroutineContext)null, (CoroutineStart)null,func2, 3, (Object)null);
	        Thread.sleep(3000L);
	     }
	  
	  
	  
	  ```
-