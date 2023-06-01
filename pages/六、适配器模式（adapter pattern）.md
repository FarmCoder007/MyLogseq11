- # 一、适配器模式概述
	- 适配器模式将某个类的接口转换成客户端期望的另一个接口表示，目的是消除由于接口不匹配所造成的类的兼容性问题。
	- 主要分为三类：类的适配器模式、对象的适配器模式、接口的适配器模式。
- # 二、现实生活中的例子
	- 如果去欧洲国家去旅游的话，他们的插座如下图最左边，是欧洲标准。而我们使用的插头如下图最右边的。因此我们的笔记本电脑，手机在当地不能直接充电。所以就需要一个插座转换器，转换器第1面插入当地的插座，第2面供我们充电，这样使得我们的插头在当地能使用。生活中这样的例子很多，手机充电器（将220v转换为5v的电压），读卡器等，其实就是使用到了适配器模式。
- #  三、适配器模式结构
	- **目标（Target）接口：**当前系统业务所期待的接口，它可以是抽象类或接口。
	- **适配者（Adaptee）类：**它是被访问和适配的现存组件库中的组件接口。
	- **适配器（Adapter）类：**它是一个转换器，通过继承或引用适配者的对象，把适配者接口转换成目标接口，让客户按目标接口的格式访问适配者。
- # 四、适配器模式的分类
  collapsed:: true
	- ## 2-1、类的适配器模式（实现目标接口+继承待适配者）实际使用少
	  collapsed:: true
		- ## 概述
			- 类适配器模式使用继承来实现适配器。适配器类同时继承目标接口（Target Interface）和被适配者类（Adaptee Class），通过继承关系使得适配器类既具有目标接口的行为，又能调用被适配者类的方法来完成适配操作。
		- ## 缺点
		  collapsed:: true
			- 违背了合成复用原则
			- 耦合度高
		- ## 示例：
		  collapsed:: true
			- 现有一台电脑只能读取SD卡（即目标接口），而要读取TF卡（被适配的类）中的内容的话就需要使用到适配器模式。
			- ## 分析
				- 1、适配器的写法：实现SD卡读取接口，继承TF读卡实现类，
				- 2、重新SD卡读写方法，内部调用TF实现类具体读取TF的方法即可
				- 这样电脑可以通过适配器来读取TF卡
			- ## 代码
				- 读取SD卡的接口和实现类
				  collapsed:: true
					- ```java
					  //SD卡的接口
					  public interface SDCard {
					      //读取SD卡方法
					      String readSD();
					      //写入SD卡功能
					      void writeSD(String msg);
					  }
					  
					  //SD卡实现类
					  public class SDCardImpl implements SDCard {
					      public String readSD() {
					          String msg = "sd card read a msg :hello word SD";
					          return msg;
					      }
					  
					      public void writeSD(String msg) {
					          System.out.println("sd card write msg : " + msg);
					      }
					  }
					  ```
				- 电脑类读卡操作，只能接收读取SD卡的接口
				  collapsed:: true
					- ```java
					  //电脑类
					  public class Computer {
					     // 只能读取SD卡接口
					      public String readSD(SDCard sdCard) {
					          if(sdCard == null) {
					              throw new NullPointerException("sd card null");
					          }
					          return sdCard.readSD();
					      }
					  }
					  ```
				- 读取TF卡接口和实现类
				  collapsed:: true
					- ```java
					  //TF卡接口
					  public interface TFCard {
					      //读取TF卡方法
					      String readTF();
					      //写入TF卡功能
					      void writeTF(String msg);
					  }
					  
					  //TF卡实现类
					  public class TFCardImpl implements TFCard {
					  
					      public String readTF() {
					          String msg ="tf card read msg : hello word tf card";
					          return msg;
					      }
					  
					      public void writeTF(String msg) {
					          System.out.println("tf card write a msg : " + msg);
					      }
					  }
					  ```
				- [[#red]]==**适配器，实现SD卡读取接口，继承TF读卡实现类**==
				  collapsed:: true
					- ```java
					  //定义适配器类（SD兼容TF）
					  public class SDAdapterTF extends TFCardImpl implements SDCard {
					  
					      public String readSD() {
					          System.out.println("adapter read tf card ");
					          return readTF();
					      }
					  
					      public void writeSD(String msg) {
					          System.out.println("adapter write tf card");
					          writeTF(msg);
					      }
					  }
					  ```
				- 测试类
				  collapsed:: true
					- 电脑直接读取SD卡
					- 电脑通过适配器读取TF卡
					- ```java
					  //测试类
					  public class Client {
					      public static void main(String[] args) {
					          Computer computer = new Computer();
					          SDCard sdCard = new SDCardImpl();
					          System.out.println(computer.readSD(sdCard));
					  
					          System.out.println("------------");
					  
					          SDAdapterTF adapter = new SDAdapterTF();
					          System.out.println(computer.readSD(adapter));
					      }
					  }
					  ```
	- ## 2-2、对象的适配器模式（实现目标接口+持有待适配者对象）[推荐]
	  collapsed:: true
		- ## 概述
			- 基本思路和类的适配器模式相同，只是将Adapter类作修改，这次不继承待适配TF实现类，而是持有TF实现类对象，以达到解决兼容性的问题
			- 适配器类：
			  collapsed:: true
				- ```JAVA
				  //创建适配器对象（SD兼容TF）
				  public class SDAdapterTF  implements SDCard {
				  
				      private TFCard tfCard;
				  
				      public SDAdapterTF(TFCard tfCard) {
				          this.tfCard = tfCard;
				      }
				  
				      public String readSD() {
				          System.out.println("adapter read tf card ");
				          return tfCard.readTF();
				      }
				  
				      public void writeSD(String msg) {
				          System.out.println("adapter write tf card");
				          tfCard.writeTF(msg);
				      }
				  }
				  ```
			- 测试类
				- ```java
				  //测试类
				  public class Client {
				      public static void main(String[] args) {
				          Computer computer = new Computer();
				          SDCard sdCard = new SDCardImpl();
				          System.out.println(computer.readSD(sdCard));
				  
				          System.out.println("------------");
				  
				          TFCard tfCard = new TFCardImpl();
				          SDAdapterTF adapter = new SDAdapterTF(tfCard);
				          System.out.println(computer.readSD(adapter));
				      }
				  }
				  ```
	- ## 2-3、接口的适配器模式
	  collapsed:: true
		- 当不希望实现一个接口中所有的方法时，可以创建一个抽象类Adapter ，实现所有方法。而此时我们只需要继承该抽象类即可。
		- 代码：
			- ```java
			  // 方法很多的接口
			  public interface Sourceable {  
			        
			      public void method1();  
			      public void method2();  
			  }  
			  
			  // 接口适配器类
			  public abstract class Wrapper2 implements Sourceable{  
			        
			      public void method1(){}  
			      public void method2(){}  
			  }  
			  
			  // 具体使用1 
			  public class SourceSub1 extends Wrapper2 {  
			      public void method1(){  
			          System.out.println("the sourceable interface's first Sub1!");  
			      }  
			  }  
			  
			  // 具体使用2
			  public class SourceSub2 extends Wrapper2 {  
			      public void method2(){  
			          System.out.println("the sourceable interface's second Sub2!");  
			      }  
			  } 
			  
			  // 测试类
			  public class WrapperTest {  
			    
			      public static void main(String[] args) {  
			          Sourceable source1 = new SourceSub1();  
			          Sourceable source2 = new SourceSub2();  
			            
			          source1.method1();  
			          source1.method2();  
			          source2.method1();  
			          source2.method2();  
			      }  
			  }  
			  测试输出：
			  
			  the sourceable interface's first Sub1!
			  the sourceable interface's second Sub2!
			  ```
- # 五、应用场景
  collapsed:: true
	- 以前开发的系统存在满足新系统功能需求的类，但其接口同新系统的接口不一致。
	- 使用第三方提供的组件，但组件接口定义和自己要求的接口定义不同。
	- ## 类的适配器模式：
		- 当希望将[[#red]]==****一个类****==转换成满足**另一个新接口**的类时，可以使用类的适配器模式，创建一个新类，继承原有的类，实现新的接口即可。
	- ## 对象的适配器模式：
		- 当希望将[[#red]]==**一个对象**==转换成满足另一个新接口的对象时，可以创建一个Wrapper类，持有原类的一个实例，在Wrapper类的方法中，调用实例的方法就行。
	- ## 接口的适配器模式：
		- 当[[#red]]==**不希望实现一个接口中所有的方法时**==，可以创建一个抽象类Wrapper，实现所有方法，我们写别的类的时候，继承抽象类即可。
- # 六、JDK源码应用适配器模式案例解析
	- ## 应用
		- `Reader`（字符流）、`InputStream`（字节流）的适配使用的是`InputStreamReader`
	- `InputStreamReader`继承自`java.io`包中的`Reader`，对他中的抽象的未实现的方法给出实现。如：
		- ```java
		  public int read() throws IOException {
		      return sd.read();
		  }
		  ​
		  public int read(char cbuf[], int offset, int length) throws IOException {
		      return sd.read(cbuf, offset, length);
		  }
		  ```
	- 如上代码中的sd（`StreamDecoder`类对象），在Sun的JDK实现中，实际的方法实现是对`sun.nio.cs.StreamDecoder`类的同名方法的调用封装。类结构图如下：
	- ![JDK源码应用适配器模式类图](https://www.panziye.com/wp-content/uploads/2022/06/2022060401032572.png)
	- **从上图可以看出：**
		- `InputStreamReader`是对同样实现了`Reader`的`StreamDecoder`的封装。
		  `StreamDecoder`不是Java SE API中的内容，是Sun JDK给出的自身实现。但我们知道他们对构造方法中的字节流类（InputStream）进行封装，并通过该类进行了字节流和字符流之间的解码转换。
	- 结论：
		- 从表层来看，`InputStreamReader`做了`InputStream`字节流类到`Reader`字符流之间的转换。而从如上Sun JDK中的实现类关系结构中可以看出，是`StreamDecoder`的设计实现在实际上采用了适配器模式。