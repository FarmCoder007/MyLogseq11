- # 一、工厂模式背景
	- 在java中，万物皆对象，这些对象都需要创建，如果创建的时候直接new该对象，就会对该对象耦合严重，假如我们要更换对象，所有new对象的地方都需要修改一遍，这显然违背了软件设计的开闭原则。
	- 如果我们使用工厂来生产对象，我们就只和工厂打交道就可以了，彻底和对象解耦，**如果要更换对象，直接在工厂里更换该对象即可，达到了与对象解耦的目的；**所以说，**工厂模式最大的优点就是：解耦**。
- # 二、适用场景
	- 建立工厂类，对实现了同一接口的一些类进行实例的创建
- # 三、写法对比
  collapsed:: true
	- # 产品角色
	  collapsed:: true
		- ## 抽象产品接口：
			- ```java
			  interface Product {
			      void operation();
			  } 
			  ```
		- ## 具体产品A
			- ```java
			  // 具体产品A
			  class ConcreteProductA implements Product {
			      @Override
			      public void operation() {
			          System.out.println("ConcreteProductA operation");
			      }
			  }
			  ```
		- ## 具体产品B
			- ```java
			  // 具体产品B
			  class ConcreteProductB implements Product {
			      @Override
			      public void operation() {
			          System.out.println("ConcreteProductB operation");
			      }
			  }
			  ```
	- # 类型一、简单工厂模式【不能用】
	  collapsed:: true
		- ### 概念
			- 简单工厂不属于23种设计模式，反而比较像是一种编程习惯。
			- 就是建立一个工厂类，对实现了同一接口的一些类进行实例的创建
		- ### 角色
		  collapsed:: true
			- 简单工厂模式的核心是一个工厂类，该工厂类根据传入的参数或条件来决定创建哪种产品对象。简单工厂模式的实现通常包括以下几个角色：
			- 抽象产品（Abstract Product）：定义产品的共同接口或抽象类。
			- 具体产品（Concrete Product）：实现抽象产品接口，具体产品是工厂类所创建的对象。
			- 简单工厂（Simple Factory）：负责创建具体产品对象的工厂类。
		- ### 工厂类:根据类型创建对应的实体
		  collapsed:: true
			- ```java
			  // 简单工厂
			  class SimpleFactory {
			      public static Product createProduct(String type) {
			          if (type.equals("A")) {
			              return new ConcreteProductA();
			          } else if (type.equals("B")) {
			              return new ConcreteProductB();
			          }
			          throw new IllegalArgumentException("Invalid product type");
			      }
			  }
			  ```
		- ### 测试类
		  collapsed:: true
			- ```java
			  // 客户端代码
			  public class Client {
			      public static void main(String[] args) {
			          Product productA = SimpleFactory.createProduct("A");
			          productA.operation();
			          
			          Product productB = SimpleFactory.createProduct("B");
			          productB.operation();
			      }
			  }
			  ```
		- ## 优缺点
		  collapsed:: true
			- 优点：
				- 封装了创建对象的过程，可以通过参数直接获取对象。
				- 把对象的创建和业务逻辑层分开，这样以后就避免了修改客户代码，如果要实现新产品直接修改工厂类，而不需要在原代码中修改，这样就降低了客户代码修改的可能性，更加容易扩展。
			- 缺点：
				- 增加新产品时还是需要修改工厂类的代码，违背了“开闭原则”。
		- ## 结论：违背了开闭原则
			- 简单工厂模式虽然能够将对象的创建逻辑封装在工厂类中，但它违背了开闭原则，因为每次新增一种产品时都需要修改工厂类的代码。
	- # 类型三、经典工厂方法模式
	  collapsed:: true
		- ## 角色
			- ## 抽象产品
				- 见开头产品角色
			- ## 具体产品
				- 见开头产品角色
			- ## 抽象工厂
			  collapsed:: true
				- ```java
				  // 抽象工厂接口
				  interface Factory {
				      Product createProduct();
				  }
				  ```
			- ## 具体工厂
			  collapsed:: true
				- ```java
				  // 具体工厂A
				  class ConcreteFactoryA implements Factory {
				      @Override
				      public Product createProduct() {
				          return new ConcreteProductA();
				      }
				  }
				  ```
				- ```java
				  // 具体工厂B
				  class ConcreteFactoryB implements Factory {
				      @Override
				      public Product createProduct() {
				          return new ConcreteProductB();
				      }
				  }
				  ```
			- ## 使用示例
			  collapsed:: true
				- ```java
				  // 客户端代码
				  public class Client {
				      public static void main(String[] args) {
				          Factory factoryA = new ConcreteFactoryA();
				          Product productA = factoryA.createProduct();
				          productA.operation();
				          
				          Factory factoryB = new ConcreteFactoryB();
				          Product productB = factoryB.createProduct();
				          productB.operation();
				      }
				  }
				  ```
		- ## 优缺点：
			- **优点：**
				- 用户只需要知道具体工厂的名称就可得到所要的产品，无须知道产品的具体创建过程；
				  在系统增加新的产品时只需要添加具体产品类和对应的具体工厂类，无须对原工厂进行任何修改，满足开闭原则；
			- **缺点：**
				- 每增加一个产品就要增加一个具体产品类和一个对应的具体工厂类，这增加了系统的复杂度。
	- # 类型三、多个工厂方法模式
	  collapsed:: true
		- ## 工厂类
			- ```java
			  // 工厂类
			  class Factory {
			      public Product createProductA() {
			          return new ConcreteProductA();
			      }
			  
			      public Product createProductB() {
			          return new ConcreteProductB();
			      }
			  }
			  ```
		- ## 测试代码
			- ```java
			  // 客户端代码
			  public class Client {
			      public static void main(String[] args) {
			          Factory factory = new Factory();
			  
			          Product productA = factory.createProductA();
			          productA.operation();
			  
			          Product productB = factory.createProductB();
			          productB.operation();
			      }
			  }
			  ```
	- # [[#red]]==**类型四、静态工厂方法模式【推荐】**==
	  collapsed:: true
		- 静态工厂方法模式是工厂方法模式的一种变体，它使用工厂类的静态方法来创建对象，无需创建具体的工厂类。
		- ## 角色
			- ## 抽象产品
				- 见开头产品角色
			- ## 具体产品
				- 见开头产品角色
			- ## 静态工厂类
				- ```java
				  // 静态工厂类
				  class StaticFactory {
				      public static Product createProductA() {
				          return new ConcreteProductA();
				      }
				      
				      public static Product createProductB() {
				          return new ConcreteProductB();
				      }
				  }
				  ```
			- ## 测试代码
				- ```java
				  // 客户端代码
				  public class Client {
				      public static void main(String[] args) {
				          Product productA = StaticFactory.createProductA();
				          productA.operation();
				          
				          Product productB = StaticFactory.createProductB();
				          productB.operation();
				      }
				  }
				  ```
- # 四、工厂方法和静态工厂方法对比
  collapsed:: true
	- 抽象程度：
		- 工厂方法模式中，客户端通过与抽象工厂接口交互来创建产品对象，具体的产品创建逻辑由具体的工厂类实现。
		- 而静态工厂方法模式中，客户端直接调用工厂类的静态方法来创建产品对象，无需实例化具体的工厂类。
	- 扩展性：
		- 工厂方法模式添加新的产品时需要创建具体的工厂类，
		- 而静态工厂方法模式通过创建新的静态工厂方法来添加新的产品，无需创建具体的工厂类，更加灵活和可扩展。
	- 客户端调用方式：
		- 工厂方法模式中，客户端通过抽象工厂接口与具体工厂类进行交互，通过工厂类的实例方法来创建产品对象。
		- 而静态工厂方法模式中，客户端直接调用工厂类的静态方法来创建产品对象。
- # 五、应用场景
	- 1、同城首页采用的静态工厂方法，ViewholderFactory,创建viewholder
	- 2、DateForamt类中的getInstance()方法使用的是工厂模式；
	- 3、Calendar类中的getInstance()方法使用的是工厂模式；
	- 4、Collection.iterator方法使用的是工厂模式；
- # 六、JDK源码解析-Collection.iterator方法
	- ```java
	  public class Demo {
	      public static void main(String[] args) {
	          List<String> list = new ArrayList<>();
	          list.add("令狐冲");
	          list.add("风清扬");
	          list.add("任我行");
	  
	          //获取迭代器对象
	          Iterator<String> it = list.iterator();
	          //使用迭代器遍历
	          while(it.hasNext()) {
	              String ele = it.next();
	              System.out.println(ele);
	          }
	      }
	  }
	  ```
	- 类图
	  collapsed:: true
		- ![collection类图.png](../assets/collection类图_1685437185291_0.png)
	- Collection接口是抽象工厂类，ArrayList是具体的工厂类；Iterator接口是抽象商品类，ArrayList类中的Iter内部类是具体的商品类。在具体的工厂类中iterator()方法创建具体的商品类的对象。