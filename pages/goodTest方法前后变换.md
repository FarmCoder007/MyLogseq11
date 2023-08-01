- ## 测试代码
	- ```kotlin
	      /**
	       *  测试多个挂起函数
	       */
	      suspend fun goodTest(): String {
	          val num = deal1()
	          val user = deal2(num)
	          val strong = deal3(user)
	          return strong
	      }
	  ```
- ## cps转换反编译后
	- ```kotlin
	     // 传入 launch函数创建的 Continuation 包裹了 completion，
	  	@Nullable
	     public final Object goodTest(@NotNull Continuation var1) {
	        Object $continuation;
	        label37: {
	           if (var1 instanceof <undefinedtype>) {
	              $continuation = (<undefinedtype>)var1;
	              if ((((<undefinedtype>)$continuation).label & Integer.MIN_VALUE) != 0) {
	                 ((<undefinedtype>)$continuation).label -= Integer.MIN_VALUE;
	                 break label37;
	              }
	           }
	  		
	           // goodTest 对Continuation 包裹1次 为 ContinuationImpl
	           $continuation = new ContinuationImpl(var1) {
	              // $FF: synthetic field
	              Object result;
	              int label;
	              Object L$0;
	  
	              @Nullable
	              public final Object invokeSuspend(@NotNull Object $result) {
	                 this.result = $result;
	                 this.label |= Integer.MIN_VALUE;
	                 return Test2.this.goodTest(this);
	              }
	           };
	        }
	  
	        Object var10000;
	        label31: {
	           Object var7;
	           label30: {
	              Object $result = ((<undefinedtype>)$continuation).result;
	              var7 = IntrinsicsKt.getCOROUTINE_SUSPENDED();
	              switch (((<undefinedtype>)$continuation).label) {
	                 case 0:
	                    ResultKt.throwOnFailure($result);
	                    ((<undefinedtype>)$continuation).L$0 = this;
	                    ((<undefinedtype>)$continuation).label = 1;
	                    var10000 = this.deal1((Continuation)$continuation);
	                    if (var10000 == var7) {
	                       return var7;
	                    }
	                    break;
	                 case 1:
	                    this = (Test2)((<undefinedtype>)$continuation).L$0;
	                    ResultKt.throwOnFailure($result);
	                    var10000 = $result;
	                    break;
	                 case 2:
	                    this = (Test2)((<undefinedtype>)$continuation).L$0;
	                    ResultKt.throwOnFailure($result);
	                    var10000 = $result;
	                    break label30;
	                 case 3:
	                    ResultKt.throwOnFailure($result);
	                    var10000 = $result;
	                    break label31;
	                 default:
	                    throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
	              }
	  
	              int num = ((Number)var10000).intValue();
	              ((<undefinedtype>)$continuation).L$0 = this;
	              ((<undefinedtype>)$continuation).label = 2;
	              var10000 = this.deal2(num, (Continuation)$continuation);
	              if (var10000 == var7) {
	                 return var7;
	              }
	           }
	  
	           User user = (User)var10000;
	           ((<undefinedtype>)$continuation).L$0 = null;
	           ((<undefinedtype>)$continuation).label = 3;
	           var10000 = this.deal3(user, (Continuation)$continuation);
	           if (var10000 == var7) {
	              return var7;
	           }
	        }
	  
	        String strong = (String)var10000;
	        return strong;
	     }
	  ```