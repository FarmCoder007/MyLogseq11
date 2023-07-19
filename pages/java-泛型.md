- # 一、什么是泛型
	- JAVA泛型(generics)是JDK5中引入的一种`参数化类型` 特性，安全机制
	- 泛型技术是给编译器使用的技术,用于编译时期。确保了类型的安全。
- # 二、[[泛型的优点]]
- # 三、使用场景
	- 当操作的引用数据类型不确定的时候。就使用<>。将要操作的引用数据类型传入即可.
	- 其实<>就是一个用于接收具体引用数据类型的参数范围。
	- > 这个类或者接口 不同的实例 的 类型可能是不同的，对于单个实例来说是确定的  ==**泛型针对的是实例**==
- # 四、[[泛型的类型擦除]]
- # 五、泛型的类型
  collapsed:: true
	- ## 1、泛型接口
	  collapsed:: true
		- 接口泛型 <T> 定义到接口名字后边
			- ```java
			  public interface Plate<T>{
			         public void set(T t);
			  }
			  ```
		-
	- ## 2、泛型类
	  collapsed:: true
		- 使用场景：什么时候用？当类中的操作的引用数据类型不确定的时候，就使用泛型来表示
		- 类中的泛型<T>定义在类名后
		- ```java
		  public class AiPlate<T> implements Plate<T>{
		  	private List<T> items = new ArrayList<T>(10);
		  }
		  ```
		-
	- ## 3、泛型方法
	  collapsed:: true
		- 方法泛型定义在 可见修饰符之后，返回值之前
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
- # 四、[[什么是参数化类型？]]
- # 六、[[泛型面试题]]
- # 七、[[协变和逆变]]
  collapsed:: true
	- ## 7-1、数组是协变
		- ```
		   // 苹果的父类水果 
		   Apple extends Fruit
		   
		   // 则 苹果数组的父类就是水果数组  这就是协变
		   Apple[] apples = new Apple[10];
		   Fruit[] fruits = new Fruit[10];
		   
		   
		  ```
- # 八、[[通配符  ？-用于泛型的实例化]]
	-
- # 九、系列文章
	- ## [泛型一、泛型类型的创建](https://blog.csdn.net/xuwb123xuwb/article/details/115520243)-记录了泛型类的扩展
	- ## [泛型二、泛型类型实例化的上界与下界](https://blog.csdn.net/xuwb123xuwb/article/details/115528458)-记录了通配符
	- ## [泛型三、泛型方法 和 类型推断](https://blog.csdn.net/xuwb123xuwb/article/details/115533636)
	- ## [泛型四、泛型的本质及使用场景](https://blog.csdn.net/xuwb123xuwb/article/details/115544416)
	- ## [泛型五、T 、＜＞ 、?、 extends、 super 使用场景总结](https://blog.csdn.net/xuwb123xuwb/article/details/115548625)
	- ## [泛型六、泛型中的 重复 和 嵌套](https://blog.csdn.net/xuwb123xuwb/article/details/115549662)
	- ## [泛型七、泛型中的类型擦除](https://blog.csdn.net/xuwb123xuwb/article/details/115550699)
	- ## [泛型八、Kotlin的泛型](https://blog.csdn.net/xuwb123xuwb/article/details/115553240)
-
- 参考
	- [[泛型介绍]]