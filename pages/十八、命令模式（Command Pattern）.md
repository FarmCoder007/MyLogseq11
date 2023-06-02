- # 一、定义
	- 将一个请求封装为一个对象，使发出请求的责任和执行请求的责任分割开。这样两者之间通过命令对象进行沟通，这样方便将命令对象进行存储、传递、调用、增加与管理。
- # 二、概述
	- 日常生活中，我们出去吃饭都会遇到下面的场景。
	  collapsed:: true
		- ![命令模式应用场景](https://www.panziye.com/wp-content/uploads/2022/06/2022060502014796.png)
	- 该场景中我们重点关注服务员向厨师递交订单，厨师根据订单来做饭菜，以前我们的传统实现方式一般是服务员对象直接聚合厨师对象，调用厨师根据订单做菜的方法，这种实现就出现服务员对象和厨师对象的强耦合，如果以后换厨师就会需要修改代码，不符合开闭原则。而，命令模式可以很好的解决该问题。
- # 三、结构
	- 命令模式包含以下主要角色：
	- **抽象命令类（Command）角色：** 定义命令的接口，声明执行的方法。
	- **具体命令（Concrete Command）角色：**具体的命令，实现命令接口；通常会持有接收者，并调用接收者的功能来完成命令要执行的操作。
	- **实现者/接收者（Receiver）角色：** 接收者，真正执行命令的对象。任何类都可能成为一个接收者，只要它能够实现命令要求实现的相应功能。
	- **调用者/请求者（Invoker）角色：** 要求命令对象执行请求，通常会持有命令对象，可以持有很多的命令对象。这个是客户端真正触发命令并要求命令执行相应操作的地方，也就是说相当于使用命令对象的入口。
- # 四、Java案例实现
	- 将上面的案例用代码实现，那我们就需要分析命令模式的角色在该案例中由谁来充当。
	- **服务员：** 就是调用者角色，由她来发起命令。
	- **资深大厨：** 就是接收者角色，真正命令执行的对象。
	- **订单： **命令中包含订单。
	- 命令模式类图如下：
	  collapsed:: true
		- ![命令模式类图](https://www.panziye.com/wp-content/uploads/2022/06/2022060502062992.png)
	- ### 订单类
		- ```java
		  public class Order {
		      //餐桌号码
		      private int diningTable;
		   
		      //所下的餐品及份数
		      private Map<String,Integer> foodDir = new HashMap<String, Integer>();
		   
		      public int getDiningTable() {
		          return diningTable;
		      }
		   
		      public void setDiningTable(int diningTable) {
		          this.diningTable = diningTable;
		      }
		   
		      public Map<String, Integer> getFoodDir() {
		          return foodDir;
		      }
		   
		      public void setFood(String name,int num) {
		          foodDir.put(name,num);
		      }
		  }
		  ```
	- ### 资深大厨类，是命令的Receiver（接收者角色）
		- ```java
		  public class SeniorChef {
		      // 根据餐品及分数制作食物
		      public void makeFood(String name,int num) {
		          System.out.println(num + "份" + name);
		      }
		  }
		  ```
	- ### 抽象命令类
		- ```java
		  public interface Command {
		      // 命令执行方法
		      void execute();
		  }
		  ```
	- ### 订单命令类，属于具体的命令类，需要聚合对象接收者和接收者依赖的操作数据
		- ```java
		  public class OrderCommand implements Command {
		   
		      //持有接收者对象
		      private SeniorChef receiver;
		      // 订单，接收者依赖的操作数据
		      private Order order;
		   
		      public OrderCommand(SeniorChef receiver, Order order) {
		          this.receiver = receiver;
		          this.order = order;
		      }
		   
		      public void execute() {
		          System.out.println(order.getDiningTable() + "桌的订单：");
		          Map<String, Integer> foodDir = order.getFoodDir();
		          //遍历map集合
		          Set<String> keys = foodDir.keySet();
		          for (String foodName : keys) {
		              receiver.makeFood(foodName, foodDir.get(foodName));
		          }
		   
		          System.out.println(order.getDiningTable() + "桌的饭准备完毕！！！");
		      }
		  }
		  ```
	- ### 服务员类，调用者（请求者）角色
		- ```java
		  public class Waitor {
		   
		      //持有多个命令对象
		      private List<Command> commands = new ArrayList<Command>();
		   
		      public void setCommand(Command cmd) {
		          //将cmd对象存储到list集合中
		          commands.add(cmd);
		      }
		   
		      //发起命令功能  喊 订单来了
		      public void orderUp() {
		          System.out.println("美女服务员：大厨，新订单来了。。。。");
		          //遍历list集合
		          for (Command command : commands) {
		             if(command != null) {
		                 command.execute();
		             }
		          }
		      }
		  }
		  ```
	- ### 测试类
		- ```java
		  public class Client {
		      public static void main(String[] args) {
		          //创建第一个订单对象
		          Order order1 = new Order();
		          order1.setDiningTable(1);
		          order1.setFood("西红柿鸡蛋面",1);
		          order1.setFood("小杯可乐",2);
		   
		          //创建第二个订单对象
		          Order order2 = new Order();
		          order2.setDiningTable(2);
		          order2.setFood("尖椒肉丝盖饭",1);
		          order2.setFood("小杯雪碧",1);
		   
		          //创建厨师对象
		          SeniorChef receiver = new SeniorChef();
		          //创建命令对象
		          OrderCommand cmd1 = new OrderCommand(receiver,order1);
		          OrderCommand cmd2 = new OrderCommand(receiver,order2);
		   
		          //创建调用者（服务员对象）
		          Waitor invoke = new Waitor();
		          invoke.setCommand(cmd1);
		          invoke.setCommand(cmd2);
		   
		          //让服务员发起命令
		          invoke.orderUp();
		      }
		  }
		  ```
	- ### 运行测试类结果：
	  collapsed:: true
		- ![命令模式案例结果](https://www.panziye.com/wp-content/uploads/2022/06/2022060502171528.png)
- # 五、优缺点
  collapsed:: true
	- ### 1，优点：
		- **降低系统的耦合度。**命令模式能将调用操作的对象与实现该操作的对象解耦。
		- **增加或删除命令非常方便。**采用命令模式增加与删除命令不会影响其他类，它满足“开闭原则”，对扩展比较灵活。
		- **可以实现宏命令。**命令模式可以与组合模式结合，将多个命令装配成一个组合命令，即宏命令。
		- **方便实现 Undo 和 Redo 操作。**命令模式可以与后面介绍的备忘录模式结合，实现命令的撤销与恢复。
	- ### 2，缺点：
		- 使用命令模式可能会导致某些系统有过多的具体命令类。
		- 系统结构更加复杂。
- # 六、使用场景
	- 系统需要将请求调用者和请求接收者解耦，使得调用者和接收者不直接交互。
	- 系统需要在不同的时间指定请求、将请求排队和执行请求。
	- 系统需要支持命令的撤销(Undo)操作和恢复(Redo)操作。
- # 七、JDK源码应用命令模式解析:Runable是一个典型命令模式
	- Runable是一个典型命令模式，Runnable担当命令的角色，Thread充当的是调用者，start方法就是其执行方法，我们一起看下源码：
	- ```java
	   
	  //命令接口(抽象命令角色)
	  public interface Runnable {
	      public abstract void run();
	  }
	  ​
	  //调用者
	  public class Thread implements Runnable {
	      private Runnable target;
	      
	      public synchronized void start() {
	          if (threadStatus != 0)
	              throw new IllegalThreadStateException();
	  ​
	          group.add(this);
	  ​
	          boolean started = false;
	          try {
	              start0();
	              started = true;
	          } finally {
	              try {
	                  if (!started) {
	                      group.threadStartFailed(this);
	                  }
	              } catch (Throwable ignore) {
	              }
	          }
	      }
	      
	      private native void start0();
	  }
	  ```
	- 会调用一个native方法`start0()`,调用系统方法，开启一个线程。而接收者是对程序员开放的，可以自己定义接收者。
	  collapsed:: true
		- ```java
		  /**
		   * jdk Runnable 命令模式
		   *      TurnOffThread ： 属于具体
		   */
		  public class TurnOffThread implements Runnable{
		       private Receiver receiver;
		      
		       public TurnOffThread(Receiver receiver) {
		          this.receiver = receiver;
		       }
		       public void run() {
		          receiver.turnOFF();
		       }
		  }
		  ```
	- 测试类，我们还需要自己定义Receiver 类，这里就不演示了
		- ```java
		  /**
		   * 测试类
		   */
		  public class Demo {
		       public static void main(String[] args) {
		           Receiver receiver = new Receiver();
		           TurnOffThread turnOffThread = new TurnOffThread(receiver);
		           Thread thread = new Thread(turnOffThread);
		           thread.start();
		       }
		  }
		  ```