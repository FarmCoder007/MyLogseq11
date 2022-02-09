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
		- 1. 检测 ‘T' 有没有做限制比如
	-
	-
-