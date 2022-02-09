- # 一、泛型的优点
	-
	- 1. 代码更健壮，约束指定的类型，在编译期可以避免类转换异常
	- 2. 代码简洁 (不需要强转)
	- 3. 代码更灵活，复用
- # 二、泛型的类型
	- 1、接口泛型
	  collapsed:: true
		- 接口泛型 <T> 定义到接口名字后边
			- ```
			  public interface Plate<T>{
			         public void set(T t);
			  }
			  ```
		-
	- 2、类泛型
	  collapsed:: true
		- 类中的泛型<T>定义在类名后
		- ```
		  public class AiPlate<T> implements Plate<T>{
		  	private List<T> items = new ArrayList<T>(10);
		  }
		  ```
		-
	- 3、方法泛型
	  collapsed:: true
		- 方法泛型定义在 可见修饰符之后，返回值之前
		- ```
		  public <T> AIPlate<T> getAiPlate(){
		         return new AIPlate<T>();
		  }
		  ```
		-
- # 三、什么是泛型
	- JAVA泛型(generics)是JDK5中引入的一种`参数化类型` 特性
- # 四、什么是参数化类型特性？
	- 把类型当参数一样传递
	- ## 例子：
	- Plate<T>中的“T” 称为类型参数
	- Plate<Banana> 中的“Banana”称为实际类型参数
	- “Plate<T>”整个称为泛型类型
	- “Plate<Banana>” 整个称为参数化的类型ParameterizedType
- # 五、泛型的类型擦除
	- ## 5-1、什么是类型擦除？
		- 泛型 ‘T’ 定义在 java 文件中比如 ‘T’ 定义为 ‘Banana’ 实际类型，编译成class字节码后，之前声明的实际类型 ‘Banana’ 被擦除
	- ## 5-2、类型擦除具体做了哪些事情
		- 1. 检测类型  ‘T' 有没有做限制 (比如 'T extends Fruit')
			- 如果没有做限制 定义的’T‘ (假如声明成’Banana‘) 会被擦除成Object
			- 如果有限制比如 'T extends Fruit'  声明的实际类型为 ’Banana‘，则会被擦除成限制的类型 ’Fruit‘
		- 2. 类型擦除后class文件中还会生成Bridge 方法 (否则 直接擦除类型为Object等，那限制的接口有未实现的方法会报错)
			- ```
			  java 文件：
			  public interface Plate <T>{
			      void set(T t);
			      T get();
			  }
			  
			  类型擦除后的class文件： 
			  public interface Plate{
			      void set(Object t);
			      Object get();
			  }
			  
			  Plate子类 AIPlate  
			  public class AIPlate <T extends java.lang.Comparable<T>> implements  Plate<T>{
			       private java.util.List<T> items;
			       public AIPlate(){  }
			       public void set(Comparable t) {   }
			       public T get() {   }
			       public java.lang.String toString(){    }
			  }
			  
			  public class AIPlate <T extends java.lang.Comparable<T>> implements  Plate<T>{
			       private java.util.List<T> items;
			       public AIPlate(){  }
			       public void set(Comparable t) {   }
			       public T get() {   }
			       public java.lang.String toString(){    }
			  }
			  
			  ```
	-
	-
-