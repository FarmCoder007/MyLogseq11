- # 一、概述
  collapsed:: true
	- 【例】通过按钮来控制一个电梯的状态，一个电梯有开门状态，关门状态，停止状态，运行状态。每一种状态改变，都有可能要根据其他状态来更新处理。例如，如果电梯门现在处于运行时状态，就不能进行开门操作，而如果电梯门是停止状态，就可以执行开门操作。
	- 传统方式实现类图如下：
	  collapsed:: true
		- ![状态模式改进前UML类图](https://www.panziye.com/wp-content/uploads/2022/06/2022060507442456.png)
	- 传统方式代码如下：
		- ### 电梯接口类
		  collapsed:: true
			- ```java
			  public interface ILift {
			      //电梯的4个状态
			      //开门状态
			      public final static int OPENING_STATE = 1;
			      //关门状态
			      public final static int CLOSING_STATE = 2;
			      //运行状态
			      public final static int RUNNING_STATE = 3;
			      //停止状态
			      public final static int STOPPING_STATE = 4;
			  ​
			      //设置电梯的状态
			      public void setState(int state);
			  ​
			      //电梯的动作
			      public void open();
			      public void close();
			      public void run();
			      public void stop();
			  }
			  ```
		- ### 电梯实现类-根据不同的状态为了保证安全只能执行对应的操作
		  collapsed:: true
			- ```java
			  public class Lift implements ILift {
			      private int state;
			  
			      @Override
			      public void setState(int state) {
			          this.state = state;
			      }
			  
			      //执行关门动作
			      @Override
			      public void close() {
			          switch (this.state) {
			              case OPENING_STATE:
			                  System.out.println("电梯关门了。。。");//只有开门状态可以关闭电梯门，可以对应电梯状态表来看
			                  this.setState(CLOSING_STATE);//关门之后电梯就是关闭状态了
			                  break;
			              case CLOSING_STATE:
			                  //do nothing //已经是关门状态，不能关门
			                  break;
			              case RUNNING_STATE:
			                  //do nothing //运行时电梯门是关着的，不能关门
			                  break;
			              case STOPPING_STATE:
			                  //do nothing //停止时电梯也是关着的，不能关门
			                  break;
			          }
			      }
			  
			      //执行开门动作
			      @Override
			      public void open() {
			          switch (this.state) {
			              case OPENING_STATE://门已经开了，不能再开门了
			                  //do nothing
			                  break;
			              case CLOSING_STATE://关门状态，门打开:
			                  System.out.println("电梯门打开了。。。");
			                  this.setState(OPENING_STATE);
			                  break;
			              case RUNNING_STATE:
			                  //do nothing 运行时电梯不能开门
			                  break;
			              case STOPPING_STATE:
			                  System.out.println("电梯门开了。。。");//电梯停了，可以开门了
			                  this.setState(OPENING_STATE);
			                  break;
			          }
			      }
			  
			      //执行运行动作
			      @Override
			      public void run() {
			          switch (this.state) {
			              case OPENING_STATE://电梯不能开着门就走
			                  //do nothing
			                  break;
			              case CLOSING_STATE://门关了，可以运行了
			                  System.out.println("电梯开始运行了。。。");
			                  this.setState(RUNNING_STATE);//现在是运行状态
			                  break;
			              case RUNNING_STATE:
			                  //do nothing 已经是运行状态了
			                  break;
			              case STOPPING_STATE:
			                  System.out.println("电梯开始运行了。。。");
			                  this.setState(RUNNING_STATE);
			                  break;
			          }
			      }
			  
			      //执行停止动作
			      @Override
			      public void stop() {
			          switch (this.state) {
			              case OPENING_STATE: //开门的电梯已经是是停止的了(正常情况下)
			                  //do nothing
			                  break;
			              case CLOSING_STATE://关门时才可以停止
			                  System.out.println("电梯停止了。。。");
			                  this.setState(STOPPING_STATE);
			                  break;
			              case RUNNING_STATE://运行时当然可以停止了
			                  System.out.println("电梯停止了。。。");
			                  this.setState(STOPPING_STATE);
			                  break;
			              case STOPPING_STATE:
			                  //do nothing
			                  break;
			          }
			      }
			  }
			  ```
		- ### 测试类
		  collapsed:: true
			- ```java
			  public class Client {
			      public static void main(String[] args) {
			  		//创建电梯对象
			          Lift lift = new Lift();
			  		//设置当前电梯的状态
			          lift.setState(ILift.RUNNING_STATE);
			  		//打开
			          lift.open();
			          lift.close();
			          lift.run();
			          lift.stop();
			      }
			  }
			  ```
		- ### 运行效果：
			- ![案例运行效果](https://www.panziye.com/wp-content/uploads/2022/06/2022060508011860.png)
		- ### 以上代码问题分析：
			- 使用了大量的switch…case这样的判断（if…else也是一样)，使程序的可阅读性变差。
			- 扩展性很差。如果新加了断电的状态，我们需要修改上面判断逻辑
- # 二、定义
	- 对有状态的对象，把复杂的“判断逻辑”提取到不同的状态对象中，允许状态对象在其内部状态发生改变时改变其行为。
- # 三、结构
	- 状态模式包含以下主要角色。
	- **环境（Context）角色：**也称为上下文，它定义了客户程序需要的接口，[[#red]]==**维护一个当前状态，并将与状态相关的操作委托给当前状态对象来处理。**==
	- **抽象状态（State）角色：**定义一个接口，用以封装环境对象中的特定状态所对应的行为。
	- **具体状态（Concrete State）角色：**实现抽象状态所对应的行为。
- # 四、Java案例实现
	- 对上述电梯的案例使用状态模式进行改进。类图如下：
	  collapsed:: true
		- ![状态模式UML类图](https://www.panziye.com/wp-content/uploads/2022/06/2022060507510735.png)
	- ### 抽象状态类
	  collapsed:: true
		- ```java
		  //抽象状态类
		  public abstract class LiftState {
		      //定义一个环境角色，也就是封装状态的变化引起的功能变化
		      protected Context context;
		  
		      public void setContext(Context context) {
		          this.context = context;
		      }
		  
		      //电梯开门动作
		      public abstract void open();
		  
		      //电梯关门动作
		      public abstract void close();
		  
		      //电梯运行动作
		      public abstract void run();
		  
		      //电梯停止动作
		      public abstract void stop();
		  }
		  ```
	- ### 开启状态类-具体状态类角色
	  collapsed:: true
		- ```java
		  //开启状态
		  public class OpenningState extends LiftState {
		  
		      //开启当然可以关闭了，我就想测试一下电梯门开关功能
		      @Override
		      public void open() {
		          System.out.println("电梯门开启...");
		      }
		  
		      @Override
		      public void close() {
		          //状态修改
		          super.context.setLiftState(Context.CLOSING_STATE);
		          //动作委托为CloseState来执行，也就是委托给了ClosingState子类执行这个动作
		          super.context.getLiftState().close();
		      }
		  
		      //电梯门不能开着就跑，这里什么也不做
		      @Override
		      public void run() {
		         //do nothing
		      }
		  
		      //开门状态已经是停止的了
		      @Override
		      public void stop() {
		        //do nothing
		      }
		  }
		  ```
	- ### 运行状态类-具体状态类角色
	  collapsed:: true
		- ```java
		  //运行状态
		  public class RunningState extends LiftState {
		  
		      //运行的时候开电梯门？你疯了！电梯不会给你开的
		      @Override
		      public void open() {
		  //do nothing
		      }
		  
		      //电梯门关闭？这是肯定了
		      @Override
		      public void close() {//虽然可以关门，但这个动作不归我执行
		  //do nothing
		      }
		  
		      //这是在运行状态下要实现的方法
		      @Override
		      public void run() {
		          System.out.println("电梯正在运行...");
		      }
		  
		      //这个事绝对是合理的，光运行不停止还有谁敢做这个电梯？！估计只有上帝了
		      @Override
		      public void stop() {
		          super.context.setLiftState(Context.STOPPING_STATE);
		          super.context.stop();
		      }
		  }
		  ```
	- ### 停止状态类-具体状态类角色
	  collapsed:: true
		- ```java
		  //停止状态
		  public class StoppingState extends LiftState {
		  
		      //停止状态，开门，那是要的！
		      @Override
		      public void open() {
		  //状态修改
		          super.context.setLiftState(Context.OPENNING_STATE);
		  //动作委托为CloseState来执行，也就是委托给了ClosingState子类执行这个动作
		          super.context.getLiftState().open();
		      }
		  
		      @Override
		      public void close() {//虽然可以关门，但这个动作不归我执行
		  //状态修改
		          super.context.setLiftState(Context.CLOSING_STATE);
		  //动作委托为CloseState来执行，也就是委托给了ClosingState子类执行这个动作
		          super.context.getLiftState().close();
		      }
		  
		      //停止状态再跑起来，正常的很
		      @Override
		      public void run() {
		  //状态修改
		          super.context.setLiftState(Context.RUNNING_STATE);
		  //动作委托为CloseState来执行，也就是委托给了ClosingState子类执行这个动作
		          super.context.getLiftState().run();
		      }
		  
		      //停止状态是怎么发生的呢？当然是停止方法执行了
		      @Override
		      public void stop() {
		          System.out.println("电梯停止了...");
		      }
		  }
		  ```
	- ### 关闭状态类-具体状态类角色
	  collapsed:: true
		- ```java
		  //关闭状态
		  public class ClosingState extends LiftState {
		  
		      @Override
		  //电梯门关闭，这是关闭状态要实现的动作
		      public void close() {
		          System.out.println("电梯门关闭...");
		      }
		  
		      //电梯门关了再打开，逗你玩呢，那这个允许呀
		      @Override
		      public void open() {
		          super.context.setLiftState(Context.OPENNING_STATE);
		          super.context.open();
		      }
		  
		          
		      //电梯门关了就跑，这是再正常不过了
		      @Override
		      public void run() {
		          super.context.setLiftState(Context.RUNNING_STATE);
		          super.context.run();
		      }
		  
		      //电梯门关着，我就不按楼层
		      @Override
		      public void stop() {
		          super.context.setLiftState(Context.STOPPING_STATE);
		          super.context.stop();
		      }
		  }
		  ```
	- ### 环境角色类
	  collapsed:: true
		- ```java
		  //环境角色
		  public class Context {
		      //定义出所有的电梯状态
		      public final static OpenningState OPENNING_STATE= new OpenningState();//开门状态，这时候电梯只能关闭
		      public final static ClosingState CLOSING_STATE= new ClosingState();//关闭状态，这时候电梯可以运行、停止和开门
		      public final static RunningState RUNNING_STATE= new RunningState();//运行状态，这时候电梯只能停止
		      public final static StoppingState STOPPING_STATE= new StoppingState();//停止状态，这时候电梯可以开门、运行
		  
		          
		      //定义一个当前电梯状态
		      private LiftState liftState;
		  
		      public LiftState getLiftState() {
		          return this.liftState;
		      }
		  
		      public void setLiftState(LiftState liftState) {
		  //当前环境改变
		          this.liftState = liftState;
		  //把当前的环境通知到各个实现类中
		          this.liftState.setContext(this);
		      }
		  
		      public void open() {
		          this.liftState.open();
		      }
		  
		      public void close() {
		          this.liftState.close();
		      }
		  
		      public void run() {
		          this.liftState.run();
		      }
		  
		      public void stop() {
		          this.liftState.stop();
		      }
		  }
		  ```
	- ### 测试类
	  collapsed:: true
		- ```java
		  //测试类
		  public class Client {
		      public static void main(String[] args) {
		  //创建环境角色对象
		          Context context = new Context();
		  //设置当前电梯装填
		          context.setLiftState(new ClosingState());
		          context.open();
		          context.run();
		          context.close();
		          context.stop();
		      }
		  }
		  ```
	- 运行测试类，测试结果如图：
		- ![状态模式案例运行结果](https://www.panziye.com/wp-content/uploads/2022/06/202206050825242.png)
- # 五、优缺点
	- 1，优点：
		- 将所有与某个状态有关的行为放到一个类中，并且可以方便地增加新的状态，只需要改变对象状态即可改变对象的行为。
		- 允许状态转换逻辑与状态对象合成一体，而不是某一个巨大的条件语句块。
	- 2，缺点：
		- 状态模式的使用必然会增加系统类和对象的个数。
		- 状态模式的结构与实现都较为复杂，如果使用不当将导致程序结构和代码的混乱。
		- 状态模式对”开闭原则”的支持并不太好。
- # 六、应用场景
	- 当一个对象的行为取决于它的状态，并且它必须在运行时根据状态改变它的行为时，就可以考虑使用状态模式。
	- 一个操作中含有庞大的分支结构，并且这些分支决定于对象的状态时。