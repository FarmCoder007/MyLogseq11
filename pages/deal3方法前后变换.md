- ## 变换前
	- ```kotlin
	      suspend fun deal3(user: User): String {
	          delay(300)
	          return "ss"
	      }
	  ```
- ## 变换后
	- ```kotlin
	     @Nullable
	     public final Object deal3(@NotNull User var1, @NotNull Continuation var2) {
	        Object $continuation;
	        label20: {
	           if (var2 instanceof <undefinedtype>) {
	              $continuation = (<undefinedtype>)var2;
	              if ((((<undefinedtype>)$continuation).label & Integer.MIN_VALUE) != 0) {
	                 ((<undefinedtype>)$continuation).label -= Integer.MIN_VALUE;
	                 break label20;
	              }
	           }
	  
	           $continuation = new ContinuationImpl(var2) {
	              // $FF: synthetic field
	              Object result;
	              int label;
	  
	              @Nullable
	              public final Object invokeSuspend(@NotNull Object $result) {
	                 this.result = $result;
	                 this.label |= Integer.MIN_VALUE;
	                 return Test2.this.deal3((User)null, this);
	              }
	           };
	        }
	  
	        Object $result = ((<undefinedtype>)$continuation).result;
	        Object var5 = IntrinsicsKt.getCOROUTINE_SUSPENDED();
	        switch (((<undefinedtype>)$continuation).label) {
	           case 0:
	              ResultKt.throwOnFailure($result);
	              ((<undefinedtype>)$continuation).label = 1;
	              if (DelayKt.delay(300L, (Continuation)$continuation) == var5) {
	                 return var5;
	              }
	              break;
	           case 1:
	              ResultKt.throwOnFailure($result);
	              break;
	           default:
	              throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
	        }
	  
	        return "ss";
	     }
	  ```