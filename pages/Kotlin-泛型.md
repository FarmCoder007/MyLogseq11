- ## 1、通配符java为？ kotlin为*
  collapsed:: true
	- ```kotlin
	  List<?> objects = new ArrayList<String>();
	   
	   
	  kotlin:
	   
	  val objects:List<*> = ArrayList<String>();
	  ```
- ## 2、kotlin泛型多范围定义
  collapsed:: true
	- ![image.png](../assets/image_1690370166662_0.png)
- ## 3、写法对比
	- ## java
	  collapsed:: true
		- ```java
		  interface Shop<T> { 
		    T buy();
		   
		    float refund(T item);
		  }
		  ```
	- ## kotlin
	  collapsed:: true
		- ```kotlin
		  class KotlinShop<T> {
		    fun buy(): T {
		      return null as T
		    }
		   
		    fun refund(item: T): Float {
		      return 1f
		    }
		  }
		  ```
- ## 4、**定义泛型上线**
	- java:**<T extends Apple>**
	- kotlin：**<T : Apple>**
- ## 5、实例化 泛型时：
  collapsed:: true
	- Java 的 <? extends> 在 Kotlin ⾥写作 <out> ； Java 的 <? super> 在 Kotlin ⾥写作 <in>
	- ## java
		- ```java
		  List<? extends Object> objects = new ArrayList<String>(); // covariant 协变
		  List<? super String> strings2 = new ArrayList<Object>(); // contravariant 逆变 / 反变
		  ```
		- java 里  <? extends>   实际作用只能调用获取的方法   不能调 加入的方法       kotlin 写out   意思往外拿
		- java 里 <? super>   实际作用只能调用加入传入的方法   不能调 获取的方法       kotlin 写in   意思往里扔
	- ## kotlin
		- ```kotlin
		   // 左边父类泛型  右边子类泛型
		   val kotlinShop: KotlinShop<out Fruit> = KotlinShop<Apple>()
		   
		   // 左边子类泛型  右边父类泛型
		   val kotlinShop2: KotlinShop<in Apple> = KotlinShop<Fruit>()
		  ```
- ## 6、kotlin 新增 out T in T
  collapsed:: true
	- ### 来在类或接⼝的声明处就限制 使⽤，这样你在使⽤时就不必再每次都写；
	- 场景： **当一个类里只有 获取的方法  根本没有加入的方法  ，那么这样用的时候 需要加多次out时候**
		- ```kotlin 
		   val kotlinShop2: KotlinShop<out Fruit> = KotlinShop<Apple>()
		   val kotlinShop3: KotlinShop<out Fruit> = KotlinShop<Apple>()
		   val kotlinShop4: KotlinShop<out Fruit> = KotlinShop<Apple>()
		   val kotlinShop5: KotlinShop<out Fruit> = KotlinShop<Apple>()
		  ```
	- ## 改造:那么可以在声明泛型的时候 直接加上out
		- ```kotlin 
		  class KotlinShop<out T : Fruit> {
		    fun buy(): T {
		      return null as T
		    }
		  }
		  ```
	- ## 使用
		- ```kotlin 
		  val kotlinShop4: KotlinShop<Fruit> = KotlinShop<Apple>()
		  ```