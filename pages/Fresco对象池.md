- 图
  collapsed:: true
	- ![image.png](../assets/image_1684400266592_0.png)
- ## 一、对象池关键点
  collapsed:: true
	- 1、返回一个等于或大于对象大小的空间，如果对象池有直接返回，没有则创建后再返回。懒加载方式
	- 2、对象池订阅了MemoryTrimmableRegistry，可以监听低内存事件，释放一部分或全部对象池对象
	- 3、内存大小三种不同概念
	  collapsed:: true
		- 逻辑大小紧紧是值的大小，例如，对于字节数组，大小就是长度。对于位图来说，大小就是像素的数量。
		- 桶形尺寸通常代表一组离散的逻辑尺寸-这样每个桶形尺寸可以容纳一个逻辑尺寸范围。例如，对于字节数组，使用桶大小的2的乘方可以让这些字节数组支持许多逻辑大小。
		- size -in-bytes就是以字节为单位的值的大小。
	- 4、复用对象池和使用完需释放对象到对象池，必须成对出现
	  对象池大小需有上限
- ## 二、Fresco对象池具体实现
  collapsed:: true
	- 关键点
	  collapsed:: true
		- 1对 象池的结构
		  final SparseArray<Bucket<V>> mBuckets;每个Bucket是一个双向链表
		- 2对象池根据配置的多个具体空间大小值实例化多个桶，每个桶初始自由链表个数为0
		- 3从对象池get空间时，对象池没有则new大小空间对象
		- 4释放对象后，将该对象放入对象池
	- 图
	  collapsed:: true
		- ![image.png](../assets/image_1684400373565_0.png)
	- Bucket
		- Bucket通过队列(LinkedList)维护了一组空闲且相同大小的值称为自由链表，通过BasePool.get(Object)请求时会找到一个合适的Bucket，如果自由链表不为空则返回一个值并从自由链表删除
		  collapsed:: true
			- ```
			    public V get() {
			      V value = mFreeList.poll();
			      if (value != null) {
			        //bucket还维护了当前“正在使用”的项(来自这个bucket)的数量,
			  	// 桶的“长度”是桶中当前正在使用的值的数量(mInUseLength)，加上自由列表的大小
			        mInUseLength++;
			      }
			      return value;
			    }
			  ```
		- 当一个值通过调用BasePool.release(Object)被释放到存储池中时，存储池会找到合适的存储池，并将该值返回给存储池的自由列表
		  collapsed:: true
			- ```
			   public void release(V value) {
			      Preconditions.checkNotNull(value);
			      ...
			        if (mInUseLength > 0) {
			          mInUseLength--;
			          mFreeList.add(value);
			        } else {
			          FLog.e(TAG, "Tried to release value %s from an empty bucket!", value);
			        }
			      }
			    }
			  ```
		- 实例化并填充mBuckets
		  collapsed:: true
			- ```
			   private void fillBuckets(SparseIntArray bucketSizes) {
			      mBuckets.clear();
			      for (int i = 0; i < bucketSizes.size(); ++i) {
			  	// 桶的每个元素空间的大小
			        final int bucketSize = bucketSizes.keyAt(i);
			  	// 每个桶自由链表长度
			        final int maxLength = bucketSizes.valueAt(i);
			        mBuckets.put(
			            bucketSize,
			  	// 一个桶内的所有元素空间都是一样大
			            new Bucket<V>(
			                getSizeInBytes(bucketSize), maxLength, 0, mPoolParams.fixBucketsReinitialization));
			      }
			    }
			  ```
		-
- ## 三、获取对象池对象
	- 关键点
	  一个软上限，达到这个上限需要调整对象池大小，将超过软上限的空间去掉，直到低于软上限空间
	  硬上限，超过直接抛异常
	  从池中获取一个新的“值”(如果可用)。在必要时分配一个新值。如果我们需要执行分配，—如果池大小超过max-size软上限，那么我们尝试修剪池的空闲部分。-如果池大小超过max-size硬上限(在调整后)，那么我们抛出一个BasePool.PoolSizeViolationException