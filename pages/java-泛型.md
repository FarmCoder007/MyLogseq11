- # 一、什么是泛型
	- JAVA泛型(generics)是JDK5中引入的一种`参数化类型` 特性，安全机制
	- 泛型技术是给编译器使用的技术,用于编译时期。确保了类型的安全。
- # 二、泛型的优点
	- id:: 643ab2eb-3b9c-4a8a-84f2-d5cc53b81b4f
	  1. 代码更健壮，约束指定的类型，将运行时期的类转换异常  转到了 编译期
	- 2. 代码简洁 (不需要强转)
	- 3. 代码更灵活，复用
- # 三、使用场景
	- <>:什么时候用？当操作的引用数据类型不确定的时候。就使用<>。将要操作的引用数据类型传入即可.
	     其实<>就是一个用于接收具体引用数据类型的参数范围。
- # 四、泛型的类型擦除
  collapsed:: true
	- ==泛型技术==是==给编译器使用==的技术,==用于编译时期==。确保了类型的安全。
	- ## 1、什么是类型擦除？
		- 运行时，会将泛型去掉，生成的class文件中是不带泛型的,这个称为泛型的擦除
		- 比如：泛型 ‘T’ 定义在 java 文件中比如 ‘T’ 定义为 ‘Banana’ 实际类型，编译成class字节码后，之前声明的实际类型 ‘Banana’ 被擦除
	- ## 2、为什么擦除呢？
		- 为了兼容运行时的类加载器。
		- 因为在类文件class中加入泛型的话，意味着之前的类加载器兼容1.5的，所以只在编译器加入了泛型机制，运行时擦除，类加载器不变。
	- ## 3、类型擦除具体做了哪些事情
	  collapsed:: true
		- 1. 检测类型  ‘T' 有没有做限制 (比如 'T extends Fruit')
			- 如果没有做限制 定义的’T‘ (假如声明成’Banana‘) 会被擦除成Object
			- 如果有限制比如 'T extends Fruit'  声明的实际类型为 ’Banana‘，则会被擦除成限制的类型 ’Fruit‘
		- 2. 类型擦除后class文件中还会生成Bridge 方法 (否则 直接擦除类型为Object等，那限制的接口有未实现的方法会报错)
			- ```
			  1、java 文件：
			  public interface Plate <T>{
			      void set(T t);
			      T get();
			  }
			  
			  2、类型擦除后的class文件： 
			  public interface Plate{
			      void set(Object t);
			      Object get();
			  }
			  
			  3、Plate子类 AIPlate  
			  public class AIPlate <T extends java.lang.Comparable<T>> implements  Plate<T>{
			       private java.util.List<T> items;
			       public AIPlate(){  }
			       public void set(Comparable t) {   }
			       public T get() {   }
			       public java.lang.String toString(){    }
			  }
			  
			  4、擦除后的 T  为 Comparable   但是implements Plate 如2里(类型擦除后的胃Object) 但是这里擦除后  set get方法里为Comparable，没有Plate中的object的set  get实现方法
			  public class AIPlate  implements  Plate{
			       private java.util.List<Comparable> items;
			       public AIPlate(){  }
			       public void set(Comparable t) {   }
			       public Comparable get() {   }
			       public java.lang.String toString(){    }
			  }
			  
			  5、所以就需要生成桥接方法
			  public class AIPlate  implements  Plate{
			       private java.util.List<Comparable> items;
			       public AIPlate(){  }
			       public void set(Comparable t) {   }
			       public Comparable get() {   }
			       public java.lang.String toString(){    }
			       
			       
			       // 桥方法 兼容AIPlate里未实现object类型的set  get方法
			       @Override
			       public synthetic bridge get(){
			       
			       }
			       @Override
			       public synthetic bridge set(Object t){
			          // 调用上边的set方法
			          set((Comparable)t)
			       }
			  }
			  ```
			- 桥方法的作用？
				- 1、类型擦除后 实现未实现的方法
				- 2、在实现的方法中 将object类型 强制成擦除后的保留类型  如上的Comparable
			-
	- ## 5-3、泛型擦除残留
		- 打开字节码文件发现里边还是含有T的，这是泛型擦除的残留，只是签名而已，还保留了定义的格式，对分析字节码有好处
	- ## 5-4、泛型运行时擦除了，为什么还能在运行时通过反射拿到类型？
		- 泛型擦除后，只是在该类的常量池里面保留了泛型的信息。所以能通过反射拿到信息
- # 五、泛型的类型
  collapsed:: true
	- ## 1、泛型接口
		- 接口泛型 <T> 定义到接口名字后边
			- ```java
			  public interface Plate<T>{
			         public void set(T t);
			  }
			  ```
		-
	- ## 2、泛型类
		- 使用场景：什么时候用？当类中的操作的引用数据类型不确定的时候，就使用泛型来表示
		- 类中的泛型<T>定义在类名后
		- ```
		  public class AiPlate<T> implements Plate<T>{
		  	private List<T> items = new ArrayList<T>(10);
		  }
		  ```
		-
	- ## 3、泛型方法
		- 方法泛型定义在 可见修饰符之后，返回值之前
		  collapsed:: true
			- ```
			  public <T> AIPlate<T> getAiPlate(){
			         return new AIPlate<T>();
			  }
			  
			  
			  public interface Plate<T>{
			         // 这种就是泛型接口里的普通方法
			         public void set(T t);
			  }
			  ```
		- 当方法==静态==时，==不能访问类上定义的泛型==。
			- 如果静态方法使用泛型，只能将泛型定义在方法上。
				- ```java
				  public class Tool<QQ>{
				  	private QQ q;
				  
				  	public QQ getObject() {
				  		return q;
				  	}
				  
				  	public void setObject(QQ object) {
				  		this.q = object;
				  	}
				  	
				  	
				  	/**
				  	 * 将泛型定义在方法上。
				  	 * @param str
				  	 */
				  	public <W> void show(W str){
				  		System.out.println("show : "+str.toString());
				  	}
				  	public void print(QQ str){
				  		System.out.println("print : "+str);
				  	}
				  	
				  	/**
				  	 * 当方法静态时，不能访问类上定义的泛型。如果静态方法使用泛型，
				  	 * 只能将泛型定义在方法上。 
				  	 * @param obj
				  	 */
				  	public static <Y> void method(Y obj){
				  		System.out.println("method:"+obj);
				  	}
				  }
				  
				  ```
- # 四、什么是参数化类型特性？
  collapsed:: true
	- 把类型当参数一样传递
	- ## 例子：
	- Plate<T>中的“T” 称为类型参数
	- Plate<Banana> 中的“Banana”称为实际类型参数
	- “Plate<T>”整个称为泛型类型
	- “Plate<Banana>” 整个称为参数化的类型ParameterizedType
- # 六、[[泛型面试题]]
- # 七、协变，逆变
  collapsed:: true
	- ## 7-1、数组是协变
		- ```
		   // 苹果的父类水果 
		   Apple extends Fruit
		   
		   // 则 苹果数组的父类就是水果数组  这就是协变
		   Apple[] apples = new Apple[10];
		   Fruit[] fruits = new Fruit[10];
		   
		   
		  ```
	-
- # 八、通配符  ？
  collapsed:: true
	- ## 概念
		- ? 代表未知类型
	- ## 8-7、 ？(通配符)和泛型有什么关系，和区别
		- 通配符让泛型的转型，更灵活
		- ## 区别
			- T泛型能代表一个类型，用于T t来接收并操作
			- ？是不能操作的？ t 这样不允许（不操作泛型就用这个）
	- ## 8-1背景： 由于六中的面试题得知，List 和 类是不变的，数组才是协变的，那么怎么让Plate<Apple>  变成 Plate<Fruit>盘子，类没协变非继承关系，不能强转，所以可以借助通配符
	  collapsed:: true
		- ```
		  public Plate<? extends Fruit> getSnack(Plate<Apple> applePlate){
		        // applePlate.set(new Banana()); // 想用放苹果的盘子 放香蕉肯定报错
		        // 但是现实中装苹果的盘子也能装香蕉和其他水果，，类没有继承关系强转也不行，就需要借助通配符？
		        // Plate<Fruit> fruitPlate = (Plate<Fruit>)applePlate;
		        
		        Plate<? extends Fruit> fruitPlate = applePlate;
		        return fruitPlate;
		  }
		  ```
	- ## 8-2、上界通配符<? extends T>：
		- 不知是否正确： 一般在存储元素的时候都是用上限，因为这样取出都是按照上限类型来运算的。不会出现类型安全隐患，体现Collection.addAll()
			- ![image.png](../assets/image_1688035180763_0.png)
			- 可以存入指定类型及其子类
		- ### 覆盖范围：
			- ![上界通配符.png](../assets/上界通配符_1644484964095_0.png)
		- ### 缺点：生产者只能取，不能存
			- ```
			  // 将苹果盘子 转为 水果盘子  
			  // 使用上界通配符 存数据时
			  public Plate<? extends Fruit> getSnack(Plate<Apple> applePlate){  
			        Plate<? extends Fruit> fruitPlate = applePlate;
			        // 使用上界通配符是能将  苹果盘子 转成 水果盘子
			        // 但是不能存放任何元素 
			        fruitPlate.set(new Apple()); // 报错
			        fruitPlate.set(new Banana()); // 报错
			        
			        // 放null 还是可以的
			        fruitPlate.set(null);
			        return fruitPlate;
			  }
			  
			  // 取数据时，只能取出 Fruit 上界类型 不能取出具体类型
			  Fruit fruit = fruitPlate.get();
			  
			  // Banana banana  = fruitPlate.get();// 取不出来
			  
			  ```
			- 不能存的解决方案：【通过反射是可以的，但是破坏了泛型的安全类型检查，如果只是自己使用偶尔可用】
				- ```
				  public Plate<? extends Fruit> getSnack(Plate<Apple> applePlate){  
				        Plate<? extends Fruit> fruitPlate = applePlate;
				        try{
				        	Method set = fruitPlate.getClass().getMethod("set",Object.class);
				          set.invoke(fruitPlate,new Banana());
				          set.invoke(fruitPlate,new Apple());
				        } catch (Exception e){}
				        return fruitPlate;
				  }
				  
				  ```
	- ## 8-3、下界通配符<? super T>:
	  collapsed:: true
		- ### 限定范围：
		  collapsed:: true
			- ![下界.png](../assets/下界_1644488998236_0.png)
		- ```
		  由上图所示  Fruit exdents Food 
		  <? super Fruit> 限定了下界，存放Fruit及以上的基类 
		  Plate<? super Fruit> lowerfruitPlate = new AIPlate<Food>();
		  lowerfruitPlate.set(new Apple());
		  lowerfruitPlate.set(new Banana());
		  // 不能存 Food  lowerfruitPlate.set(new Food());
		  
		  // 取出来 泛型信息丢失了，只能用Object存放
		  Object newFruit2 = lowerfruitPlate.get();
		  ```
		- ### 缺点：只能存，不能取
			-
		-
		-
	- ## 8-4、非限定的通配符 Plate<?> 其实就是 Plate<? extends Object>
	  collapsed:: true
		- 无限定的通配符，没有extends 和 super，只是一个？
		- 作用：既不能读也不能写，只做类型检查
	- ## 8-5、通配符继承规则
	  collapsed:: true
		-
	-
	- ## 8-6、通配符PECS (Producer extends  Consumer super)原则
	  collapsed:: true
		- ### 如果只从集合中获取类型T，使用<? extends T> 通配符，即Producer extends  从生产者取
		- ### 如果只需要将类型T存放集合中，使用<？super T>通配符，往消费者存
		- ### 如果既要存，又要取，则不使用任何通配符
		-
		- ### 为何要用PECS原则 ？
			- 提高API灵活性
		- ### <?> 既不能存也不能取，只有类型检查的作用
	-
	-
- # 九、系列文章
	- ## [泛型一、泛型类型的创建](https://blog.csdn.net/xuwb123xuwb/article/details/115520243)
	- ## [泛型二、泛型类型实例化的上界与下界](https://blog.csdn.net/xuwb123xuwb/article/details/115528458)
	- ## [泛型三、泛型方法 和 类型推断](https://blog.csdn.net/xuwb123xuwb/article/details/115533636)
	- ## [泛型四、泛型的本质及使用场景](https://blog.csdn.net/xuwb123xuwb/article/details/115544416)
	- ## [泛型五、T 、＜＞ 、?、 extends、 super 使用场景总结](https://blog.csdn.net/xuwb123xuwb/article/details/115548625)
	- ## [泛型六、泛型中的 重复 和 嵌套](https://blog.csdn.net/xuwb123xuwb/article/details/115549662)
	- ## [泛型七、泛型中的类型擦除](https://blog.csdn.net/xuwb123xuwb/article/details/115550699)
	- ## [泛型八、Kotlin的泛型](https://blog.csdn.net/xuwb123xuwb/article/details/115553240)
-
- 参考
	- [[泛型介绍]]