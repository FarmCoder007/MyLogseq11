- # 一、概述
  collapsed:: true
	- 在现实生活中，常常会出现这样的事例：
	- 请假
		- 一个请求有多个对象可以处理，但每个对象的处理条件或权限不同。例如，公司员工请假，可批假的领导有部门负责人、副总经理、总经理等，但每个领导能批准的天数不同，员工必须根据自己要请假的天数去找不同的领导签名，也就是说员工必须记住每个领导的姓名、电话和地址等信息，这增加了难度。
		-
	- 找领导出差报销
	- “击鼓传花”游戏。
- # 二、定义
	- 责任链模式，又名职责链模式，为了避免请求发送者与多个请求处理者耦合在一起，将所有请求的处理者通过前一对象记住其下一个对象的引用而连成一条链；当有请求发生时，可将请求沿着这条链传递，直到有对象处理它为止。
- # 三、结构
	- **抽象处理者（Handler）角色：**定义一个处理请求的接口，包含抽象处理方法和一个后继连接。
	- **具体处理者（Concrete Handler）角色：**实现抽象处理者的处理方法，判断能否处理本次请求，如果可以处理请求则处理，否则将该请求转给它的后继者。
	- **客户类（Client）角色：**创建处理链，并向链头的具体处理者对象提交请求，它不关心处理细节和请求的传递过程。
- # 四、Java案例实现
  collapsed:: true
	- 现需要开发一个请假流程控制系统。请假一天以下的假只需要小组长同意即可；请假1天到3天的假还需要部门经理同意；请求3天到7天还需要总经理同意才行。
	  collapsed:: true
		- ![责任链模式类图](https://www.panziye.com/wp-content/uploads/2022/06/2022060502533545.png)
	- ## 请假类
		- ```java
		  //请假条
		  public class LeaveRequest {
		      private String name;//姓名
		      private int num;//请假天数
		      private String content;//请假内容
		  ​
		      public LeaveRequest(String name, int num, String content) {
		          this.name = name;
		          this.num = num;
		          this.content = content;
		      }
		  ​
		      public String getName() {
		          return name;
		      }
		  ​
		      public int getNum() {
		          return num;
		      }
		  ​
		      public String getContent() {
		          return content;
		      }
		   
		      // 重写toString
		      @Override
		      public String toString() {
		          return this.name+ "请假" + this.name + "天，" + this.content + "。"
		      }
		  }
		  ```
	- ## 抽象处理类
	  collapsed:: true
		- ```java
		  //处理者抽象类
		  public abstract class Handler {
		   
		      protected final static int NUM_ONE = 1;
		      protected final static int NUM_THREE = 3;
		      protected final static int NUM_SEVEN = 7;
		   
		      //该领导处理的请求天数区间
		      private int numStart;
		      private int numEnd;
		   
		      //声明后续者（声明上级领导）
		      private Handler nextHandler;
		   
		      public Handler(int numStart) {
		          this.numStart = numStart;
		      }
		   
		      public Handler(int numStart, int numEnd) {
		          this.numStart = numStart;
		          this.numEnd = numEnd;
		      }
		   
		      //设置上级领导对象
		      public void setNextHandler(Handler nextHandler) {
		          this.nextHandler = nextHandler;
		      }
		   
		      //各级领导处理请求条的方法
		      protected abstract void handleLeave(LeaveRequest leave);
		   
		      //提交请求条
		      public final void submit(LeaveRequest leave) {
		          //该领导进行审批
		          this.handleLeave(leave);
		          if(this.nextHandler != null && leave.getNum() > this.numEnd) {
		              //提交给上级领导进行审批
		              this.nextHandler.submit(leave);
		          } else {
		              System.out.println("流程结束！");
		          }
		      }
		  }
		  ```
	- ## 小组长类-具体处理者角色
	  collapsed:: true
		- ```java
		  public class GroupLeader extends Handler {
		   
		      public GroupLeader() {
		          super(0,Handler.NUM_ONE);
		      }
		   
		      protected void handleLeave(LeaveRequest leave) {
		          System.out.println(leave);
		          System.out.println("小组长审批：同意");
		      }
		  }
		  ```
	- ## 部门经理类-具体处理者角色
	  collapsed:: true
		- ```java
		  public class Manager extends Handler {
		   
		      public Manager() {
		          super(Handler.NUM_ONE,Handler.NUM_THREE);
		      }
		   
		      protected void handleLeave(LeaveRequest leave) {
		          System.out.println(leave);
		          System.out.println("部门经理审批：同意");
		      }
		  }
		  ```
	- ## 总经理类-具体处理者角色
	  collapsed:: true
		- ```java
		  public class GeneralManager extends Handler {
		   
		      public GeneralManager() {
		          super(Handler.NUM_THREE,Handler.NUM_SEVEN);
		      }
		   
		      protected void handleLeave(LeaveRequest leave) {
		          System.out.println(leave);
		          System.out.println("总经理审批：同意");
		      }
		  }
		  ```
	- ## 测试类
		- ```java
		  public class Client {
		      public static void main(String[] args) {
		          //创建一个请假条对象
		          LeaveRequest leave = new LeaveRequest("小明",2,"身体不适");
		   
		          //创建各级领导对象
		          GroupLeader groupLeader = new GroupLeader();
		          Manager manager = new Manager();
		          GeneralManager generalManager = new GeneralManager();
		   
		          //设置处理者链
		          groupLeader.setNextHandler(manager);
		          manager.setNextHandler(generalManager);
		   
		   
		          //小明提交请假申请
		          groupLeader.submit(leave);
		      }
		  }
		  ```
		- 这里演示了请假2天，由小组长审批完后，交由部门经理再审批，审批完结束流程，无需再向总经理审批。效果如下图：
		- ![责任链模式案例演示结果](https://www.panziye.com/wp-content/uploads/2022/06/2022060503081411.png)
- # 五、责任链模式优缺点
  collapsed:: true
	- ### 1，优点：
		- **降低了对象之间的耦合度：**该模式降低了请求发送者和接收者的耦合度。
		- **增强了系统的可扩展性：**可以根据需要增加新的请求处理类，满足开闭原则。
		- **增强了给对象指派职责的灵活性：**当工作流程发生变化，可以动态地改变链内的成员或者修改它们的次序，也可动态地新增或者删除责任。
		- **责任链简化了对象之间的连接：**一个对象只需保持一个指向其后继者的引用，不需保持其他所有处理者的引用，这避免了使用众多的 if 或者 if···else 语句。
		- **责任分担：**每个类只需要处理自己该处理的工作，不能处理的传递给下一个对象完成，明确各类的责任范围，符合类的单一职责原则。
	- ### 2，缺点：
		- 不能保证每个请求一定被处理。由于一个请求没有明确的接收者，所以不能保证它一定会被处理，该请求可能一直传到链的末端都得不到处理。
		- 对比较长的职责链，请求的处理可能涉及多个处理对象，系统性能将受到一定影响。
		- 职责链建立的合理性要靠客户端来保证，增加了客户端的复杂性，可能会由于职责链的错误设置而导致系统出错，如可能会造成循环调用。
- # 六、责任链模式源码应用解析
  collapsed:: true
	- 在javaWeb应用开发中，`FilterChain`是职责链（过滤器）模式的典型应用，以下是Filter的模拟实现分析:
	- 1、模拟web请求Request以及web响应Response
	  collapsed:: true
		- ```java
		  	 
		  public interface Request{
		   
		  }
		  ​
		  public interface Response{
		   
		  }
		  ```
	- 2、模拟web过滤器Filter
	  collapsed:: true
		- ```java
		   public interface Filter {
		      public void doFilter(Request req,Response res,FilterChain c);
		   }
		  ```
	- 3、模拟实现具体过滤器
	  collapsed:: true
		- ```java
		  public class FirstFilter implements Filter {
		      @Override
		      public void doFilter(Request request, Response response, FilterChain chain) {
		  ​
		          System.out.println("过滤器1 前置处理");
		  ​
		          // 先执行所有request再倒序执行所有response
		          chain.doFilter(request, response);
		  ​
		          System.out.println("过滤器1 后置处理");
		      }
		  }
		  ​
		  public class SecondFilter  implements Filter {
		      @Override
		      public void doFilter(Request request, Response response, FilterChain chain) {
		  ​
		          System.out.println("过滤器2 前置处理");
		  ​
		          // 先执行所有request再倒序执行所有response
		          chain.doFilter(request, response);
		  ​
		          System.out.println("过滤器2 后置处理");
		      }
		  }
		  ```
	- 4、模拟实现过滤器链FilterChain
	  collapsed:: true
		- ```java
		  public class FilterChain {
		  ​
		      private List<Filter> filters = new ArrayList<Filter>();
		  ​
		      private int index = 0;
		  ​
		      // 链式调用
		      public FilterChain addFilter(Filter filter) {
		          this.filters.add(filter);
		          return this;
		      }
		  ​
		      public void doFilter(Request request, Response response) {
		          if (index == filters.size()) {
		              return;
		          }
		          Filter filter = filters.get(index);
		          index++;
		          filter.doFilter(request, response, this);
		      }
		  }
		  ```
	- 5、测试类
		- ```java
		  public class Client {
		      public static void main(String[] args) {
		          Request  req = null;
		          Response res = null ;
		  ​
		          FilterChain filterChain = new FilterChain();
		          filterChain.addFilter(new FirstFilter()).addFilter(new SecondFilter());
		          filterChain.doFilter(req,res);
		      }
		  }
		  ```
		- 运行效果和我们javaweb中的效果一样，要想理解为什么打印顺序会这样，需要重点去理解`FilterChain`类中`doFilter`方法：
			- ![责任链模式模拟过滤器链案例](https://www.panziye.com/wp-content/uploads/2022/06/2022060503182525.png)
- # 七、okhttp的拦截器就是责任链