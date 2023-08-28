# 一、适用场景
collapsed:: true
	- [[#red]]==**建造者模式的必要条件是需要创建一个复杂对象，并且希望将对象的构建过程与最终对象的表示分离**==
	-
	- 当创建复杂对象的算法步骤比较固定，但各个步骤的实现可以有所差异时，可以使用建造者模式。==**它可以将对象的构建过程和表示分离，使得构建过程可以灵活地组合和变化，同时保持一致的构建过程。**==
	- ==**当需要创建的对象具有复杂的内部结构，并且需要一步步地构建时**==，建造者模式可以提供一种更好的方式来创建对象。
	- 当创建过程需要多个步骤或者多个部件，并且这些步骤或部件之间存在一定的依赖关系时，建造者模式可以有效地管理和组织这些步骤或部件的创建过程。
- # 二、注意
  collapsed:: true
	- 链式调用不是建造者模式必须的条件
- # 类图
  collapsed:: true
	- ![构建者模式类图.png](../assets/构建者模式类图_1685417852753_0.png)
- # 三、举例，
  collapsed:: true
	- >电脑主机是一个复杂对象，需要多个组件对象，经过一定的组装过程才能装配出来，以下几段，说明可以结合电脑组装过程进行理解
	- 1）分离了部件的构造(由Builder来负责)和装配(由Director负责)。 从而可以构造出复杂的对象。这个模式适用于：某个对象的构建过程复杂的情况。
	- 2）由于实现了构建和装配的解耦。不同的构建器，相同的装配，也可以做出不同的对象；相同的构建器，不同的装配顺序也可以做出不同的对象。也就是实现了构建算法、装配算法的解耦，实现了更好的复用。
	- 3）建造者模式可以将部件和其组装过程分开，一步一步创建一个复杂的对象。用户只需要指定复杂对象的类型就可以得到该对象，而无须知道其内部的具体构造细节。
- # 四、创建共享单车实例
  collapsed:: true
	- 生产自行车是一个复杂的过程，它包含了车架，车座等组件的生产。而车架又有碳纤维，铝合金等材质的，车座有橡胶，真皮等材质。对于自行车的生产就可以使用建造者模式。
	- 这里Bike是产品，包含车架，车座等组件；Builder是抽象建造者，MobikeBuilder和OfoBuilder是具体的建造者；Director是指挥者。类图如下：
		- ![共享单车构建者模式.png](../assets/共享单车构建者模式_1685439055594_0.png)
- # 五、角色
  collapsed:: true
	- ## 抽象建造者类(Builder)：这个接口规定要实现复杂对象的那些部分的创建，并不涉及具体的部件对象的创建。
	  collapsed:: true
		- ```java
		  // 抽象 builder 类
		  public abstract class Builder {
		  
		      protected Bike mBike = new Bike();
		  
		      public abstract void buildFrame();
		      public abstract void buildSeat();
		      public abstract Bike createBike();
		  }
		  ```
	- ## 具体建造者类（ConcreteBuilder）：实现 Builder 接口，完成复杂产品的各个部件的具体创建方法。在构造过程完成后，提供产品的实例。
	  collapsed:: true
		- ```java
		  
		  //摩拜单车Builder类
		  public class MobikeBuilder extends Builder {
		  
		      @Override
		      public void buildFrame() {
		          mBike.setFrame("铝合金车架");
		      }
		  
		      @Override
		      public void buildSeat() {
		          mBike.setSeat("真皮车座");
		      }
		  
		      @Override
		      public Bike createBike() {
		          return mBike;
		      }
		  }
		  
		  
		  //ofo单车Builder类
		  public class OfoBuilder extends Builder {
		  
		      @Override
		      public void buildFrame() {
		          mBike.setFrame("碳纤维车架");
		      }
		  
		      @Override
		      public void buildSeat() {
		          mBike.setSeat("橡胶车座");
		      }
		  
		      @Override
		      public Bike createBike() {
		          return mBike;
		      }
		  }
		  ```
	- ## 产品类（Product）：要创建的复杂对象。
	  collapsed:: true
		- ```java
		  //自行车类
		  public class Bike {
		      private String frame;
		      private String seat;
		  
		      public String getFrame() {
		          return frame;
		      }
		  
		      public void setFrame(String frame) {
		          this.frame = frame;
		      }
		  
		      public String getSeat() {
		          return seat;
		      }
		  
		      public void setSeat(String seat) {
		          this.seat = seat;
		      }
		  }
		  ```
	- ## 指挥者类（Director）：负责控制具体建造者的构建过程
	  collapsed:: true
		- 调用具体建造者来创建复杂对象的各个部分，在指导者中不涉及具体产品的信息，只负责保证对象各部分完整创建或按某种顺序创建。
		- ```java
		  //指挥者类
		  public class Director {
		      private Builder mBuilder;
		  
		      public Director(Builder builder) {
		          mBuilder = builder;
		      }
		  
		      public Bike construct() {
		          mBuilder.buildFrame();
		          mBuilder.buildSeat();
		          return mBuilder.createBike();
		      }
		  }
		  ```
	- # 测试
	  collapsed:: true
		- ```java
		  //测试类
		  public class Client {
		      public static void main(String[] args) {
		          showBike(new OfoBuilder());
		          showBike(new MobikeBuilder());
		      }
		      private static void showBike(Builder builder) {
		          Director director = new Director(builder);
		          Bike bike = director.construct();
		          System.out.println(bike.getFrame());
		          System.out.println(bike.getSeat());
		      }
		  }
		  ```
- # 六、优缺点
  collapsed:: true
	- ## 优点：
		- 1）建造者模式的封装性很好。
			- 使用建造者模式可以有效的封装变化，在使用建造者模式的场景中，一般产品类和建造者类是比较稳定的，因此，将主要的业务逻辑封装在指挥者类中对整体而言可以取得比较好的稳定性。
		- 2）在建造者模式中，客户端[[#red]]==**不必知道产品内部组成的细节**==，将产品本身与产品的创建过程解耦，使得相同的创建过程可以创建不同的产品对象。
			- 可以更加精细地控制产品的创建过程 。将复杂产品的创建步骤分解在不同的方法中，使得创建过程更加清晰，也更方便使用程序来控制创建过程。
		- 3）建造者模式很容易进行扩展。
			- 如果有新的需求，通过实现一个新的建造者类就可以完成，基本上不用修改之前已经测试通过的代码，因此也就不会对原有功能引入风险。符合开闭原则。
	- ## 缺点：
		- 造者模式所创建的产品一般具有较多的共同点，其组成部分相似，==**如果产品之间的差异性很大，则不适合使用建造者模式，因此其使用范围受到一定的限制。**==
- # 七、应用场景
  collapsed:: true
	- 建造者（Builder）模式创建的是复杂对象，其产品的各个部分经常面临着剧烈的变化，但将它们组合在一起的算法却相对稳定，所以它通常在以下场合使用。
	- 1）[[#red]]==**创建的对象较复杂，由多个部件构成，**==各部件面临着复杂的变化，但构件间的建造顺序是稳定的。
	- 2）创建复杂对象的算法独立于该对象的组成部分以及它们的装配方式，即产品的构建过程和最终的表示是独立的。
- # 八、建造者模式结合链式调用
	- 就是当一个[[#red]]==**类构造器需要传入很多参数时**==，如果创建这个类的实例，代码可读性会非常差，而且很容易引入错误，此时就可以利用建造者模式进行重构
	- ## 重构前
	  collapsed:: true
		- ```java
		  public class Phone {
		      private String cpu;
		      private String screen;
		      private String memory;
		      private String mainboard;
		  
		      public Phone(String cpu, String screen, String memory, String mainboard) {
		          this.cpu = cpu;
		          this.screen = screen;
		          this.memory = memory;
		          this.mainboard = mainboard;
		      }
		  
		      public String getCpu() {
		          return cpu;
		      }
		  
		      public void setCpu(String cpu) {
		          this.cpu = cpu;
		      }
		  
		      public String getScreen() {
		          return screen;
		      }
		  
		      public void setScreen(String screen) {
		          this.screen = screen;
		      }
		  
		      public String getMemory() {
		          return memory;
		      }
		  
		      public void setMemory(String memory) {
		          this.memory = memory;
		      }
		  
		      public String getMainboard() {
		          return mainboard;
		      }
		  
		      public void setMainboard(String mainboard) {
		          this.mainboard = mainboard;
		      }
		  
		      @Override
		      public String toString() {
		          return "Phone{" +
		                  "cpu='" + cpu + '\'' +
		                  ", screen='" + screen + '\'' +
		                  ", memory='" + memory + '\'' +
		                  ", mainboard='" + mainboard + '\'' +
		                  '}';
		      }
		  }
		  
		  public class Client {
		      public static void main(String[] args) {
		          //构建Phone对象
		          Phone phone = new Phone("intel","三星屏幕","金士顿","华硕");
		          System.out.println(phone);
		      }
		  }
		  ```
	- ## 重构后链式调用
		- ```java
		  public class Phone {
		  
		      private String cpu;
		      private String screen;
		      private String memory;
		      private String mainboard;
		  
		      private Phone(Builder builder) {
		          cpu = builder.cpu;
		          screen = builder.screen;
		          memory = builder.memory;
		          mainboard = builder.mainboard;
		      }
		  
		      public static final class Builder {
		          private String cpu;
		          private String screen;
		          private String memory;
		          private String mainboard;
		  
		          public Builder() {}
		  
		          public Builder cpu(String val) {
		              cpu = val;
		              return this;
		          }
		          public Builder screen(String val) {
		              screen = val;
		              return this;
		          }
		          public Builder memory(String val) {
		              memory = val;
		              return this;
		          }
		          public Builder mainboard(String val) {
		              mainboard = val;
		              return this;
		          }
		          public Phone build() {
		              return new Phone(this);}
		      }
		      @Override
		      public String toString() {
		          return "Phone{" +
		                  "cpu='" + cpu + '\'' +
		                  ", screen='" + screen + '\'' +
		                  ", memory='" + memory + '\'' +
		                  ", mainboard='" + mainboard + '\'' +
		                  '}';
		      }
		  }
		  
		  public class Client {
		      public static void main(String[] args) {
		          Phone phone = new Phone.Builder()
		                  .cpu("intel")
		                  .mainboard("华硕")
		                  .memory("金士顿")
		                  .screen("三星")
		                  .build();
		          System.out.println(phone);
		      }
		  }
		  ```
- # 九、[[什么时候使用建造者模式？]]
-