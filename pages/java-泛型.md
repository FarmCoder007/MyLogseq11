- # 一、泛型的优点
	-
	- 1. 约束指定的类型，在编译期可以避免类转换异常，且不需要强转
	  2.
	-
- # 二、泛型的类型
	- 1、接口泛型
		- 接口泛型 <T> 定义到接口名字后边
			- ```
			        public interface Plate<T>{
			            public void set(T t);
			        }
			  ```
		-
	- 2、类泛型
		- 类中的泛型<T>定义在类名后
		- ```
		  public class AiPlate<T> implements Plate<T>{
		  
		  private List<T> items = 
		  }
		  ```
		-
	-
	- 3、方法泛型
-