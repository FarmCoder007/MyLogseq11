- # 适用场景
	- 建立工厂类，对实现了同一接口的一些类进行实例的创建
- # 产品角色
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
- # 工厂方法和静态工厂方法对比
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
- # 应用场景
	- 1、同城首页采用的静态工厂方法，ViewholderFactory,创建viewholder
-
-