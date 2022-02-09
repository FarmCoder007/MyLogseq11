- # 一、泛型的优点
	-
	- 1. 代码更健壮，约束指定的类型，在编译期可以避免类转换异常
	- 2. 代码简洁 (不需要强转)
	- 3. 代码更灵活，复用
	-
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
	- JAVA泛型(generics)是JDK5中引入的一种
-