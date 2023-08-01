- ## 变换前
	- ```kotlin
	      suspend fun deal1(): Int {
	          delay(22)
	          return 2
	      }
	  
	  ```
- ## 转换后
	- ```kotlin
	     @Nullable
	     public final Object deal1(@NotNull Continuation var1) {
	        Object $continuation;
	        label20: {
	           if (var1 instanceof <undefinedtype>) {
	              $continuation = (<undefinedtype>)var1;
	              if ((((<undefinedtype>)$continuation).label & Integer.MIN_VALUE) != 0) {
	                 ((<undefinedtype>)$continuation).label -= Integer.MIN_VALUE;
	                 break label20;
	              }
	           }
	  
	           $continuation = new ContinuationImpl(var1) {
	              // $FF: synthetic field
	              Object result;
	              int label;
	  
	              @Nullable
	              public final Object invokeSuspend(@NotNull Object $result) {
	                 this.result = $result;
	                 this.label |= Integer.MIN_VALUE;
	                 return Test2.this.deal1(this);
	              }
	           };
	        }
	  
	        Object $result = ((<undefinedtype>)$continuation).result;
	        Object var4 = IntrinsicsKt.getCOROUTINE_SUSPENDED();
	        switch (((<undefinedtype>)$continuation).label) {
	           case 0:
	              ResultKt.throwOnFailure($result);
	              ((<undefinedtype>)$continuation).label = 1;
	              if (DelayKt.delay(22L, (Continuation)$continuation) == var4) {
	                 return var4;
	              }
	              break;
	           case 1:
	              ResultKt.throwOnFailure($result);
	              break;
	           default:
	              throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
	        }
	  
	        return Boxing.boxInt(2);
	     }
	  ```