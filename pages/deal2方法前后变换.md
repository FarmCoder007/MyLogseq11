- ## 变换前
	- ```kotlin
	      suspend fun deal2(num: Int): User {
	          val user = User(11, "1")
	          delay(2000)
	          return user
	      }
	  ```
- ## 变换后
	- ```kotlin
	     @Nullable
	     public final Object deal2(int var1, @NotNull Continuation var2) {
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
	              Object L$0;
	  
	              @Nullable
	              public final Object invokeSuspend(@NotNull Object $result) {
	                 this.result = $result;
	                 this.label |= Integer.MIN_VALUE;
	                 return Test2.this.deal2(0, this);
	              }
	           };
	        }
	  
	        Object $result = ((<undefinedtype>)$continuation).result;
	        Object var6 = IntrinsicsKt.getCOROUTINE_SUSPENDED();
	        User user;
	        switch (((<undefinedtype>)$continuation).label) {
	           case 0:
	              ResultKt.throwOnFailure($result);
	              user = new User(11, "1");
	              ((<undefinedtype>)$continuation).L$0 = user;
	              ((<undefinedtype>)$continuation).label = 1;
	              if (DelayKt.delay(2000L, (Continuation)$continuation) == var6) {
	                 return var6;
	              }
	              break;
	           case 1:
	              user = (User)((<undefinedtype>)$continuation).L$0;
	              ResultKt.throwOnFailure($result);
	              break;
	           default:
	              throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
	        }
	  
	        return user;
	     }
	  ```