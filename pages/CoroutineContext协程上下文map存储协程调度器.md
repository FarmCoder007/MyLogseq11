- 协程上下文CoroutineContext中存储的协程调度器CoroutineDispatcher
- `Key`是`ContinuationInterceptor`，且默认会设置为`Dispatchers.Default`。
- 详细见
	- ```kotlin
	  public interface ContinuationInterceptor : CoroutineContext.Element {
	      /**
	       * The key that defines *the* context interceptor.
	       */
	      companion object Key : CoroutineContext.Key<ContinuationInterceptor>
	  ```