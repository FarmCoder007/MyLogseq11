- # 一、原型模式概述（克隆对象）
  collapsed:: true
	- 所谓原型模式，即：用一个已经创建的实例作为原型，通过[[#red]]==**复制该原型对象**==来创建一个和原型对象相同的新对象。
- # 二、原型模式结构
  collapsed:: true
	- 原型模式包含如下角色：
		- 抽象原型类：规定了具体原型对象必须实现的的 clone() 方法。
			- 就是Cloneable接口，要想实现原型模式必须实现该接口
		- 具体原型类：实现抽象原型类的 clone() 方法，它是可被复制的对象。
		- 访问类：使用具体原型类中的 clone() 方法来复制新的对象。
	- 类图：
		- ![原型模式类图.png](../assets/原型模式类图_1685443220797_0.png)
- # 三、实现（深克隆、浅克隆）
  collapsed:: true
	- ## 根据对象的变量是否也被克隆划分 深克隆，浅克隆
	- ## 深克隆（对象流序列化）
	  collapsed:: true
		- ## 概念：
			- 创建一个新对象，属性中引用的其他对象也会被克隆，不再指向原有对象地址。
		- ## 案例：
			- 将上面的“三好学生”奖状的案例中Citation类的name属性修改为Student类型的属性。代码如下：
			  collapsed:: true
				- ```java
				  //奖状类
				  public class Citation implements Cloneable {
				      private Student stu;
				  
				      public Student getStu() {
				          return stu;
				      }
				  
				      public void setStu(Student stu) {
				          this.stu = stu;
				      }
				  
				      void show() {
				          System.out.println(stu.getName() + "同学：在2020学年第一学期中表现优秀，被评为三好学生。特发此状！");
				      }
				  
				      @Override
				      public Citation clone() throws CloneNotSupportedException {
				          return (Citation) super.clone();
				      }
				  }
				  
				  //学生类
				  public class Student {
				      private String name;
				      private String address;
				  
				      public Student(String name, String address) {
				          this.name = name;
				          this.address = address;
				      }
				  
				      public Student() {
				      }
				  
				      public String getName() {
				          return name;
				      }
				  
				      public void setName(String name) {
				          this.name = name;
				      }
				  
				      public String getAddress() {
				          return address;
				      }
				  
				      public void setAddress(String address) {
				          this.address = address;
				      }
				  }
				  
				  //测试类
				  public class CitationTest {
				      public static void main(String[] args) throws CloneNotSupportedException {
				  
				          Citation c1 = new Citation();
				          Student stu = new Student("张三", "西安");
				          c1.setStu(stu);
				  
				          //复制奖状
				          Citation c2 = c1.clone();
				          //获取c2奖状所属学生对象
				          Student stu1 = c2.getStu();
				          stu1.setName("李四");
				  
				          //判断stu对象和stu1对象是否是同一个对象
				          System.out.println("stu和stu1是同一个对象？" + (stu == stu1));
				  
				          c1.show();
				          c2.show();
				      }
				  }
				  ```
			- 结果：
			  collapsed:: true
				- ![原型模式结果.png](../assets/原型模式结果_1685454200096_0.png)
				- 从运行结果中，我们可以得知`stu`和`stu1`是同一个对象，也就是说，浅拷贝不会针对非基本类型进行拷贝，只会仍指向原有属性所指向的对象的内存地址。
			- stu对象和stu1对象是同一个对象，就会产生将stu1对象中name属性值改为“李四”，两个Citation（奖状）对象中显示的都是李四。这就是浅克隆的效果，对具体原型类（Citation）中的引用类型的属性进行引用的复制。这种情况需要使用深克隆，而进行深克隆需要使用对象流。代码如下：
				- >Citation类和Student类必须实现Serializable接口，否则会抛NotSerializableException异常。
				- ```java
				   
				  public class CitationTest1 {
				      public static void main(String[] args) throws Exception {
				          Citation c1 = new Citation();
				          Student stu = new Student("张三", "西安");
				          c1.setStu(stu);
				  
				          //创建对象输出流对象
				          ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("C:\\Users\\Think\\Desktop\\b.txt"));
				          //将c1对象写出到文件中
				          oos.writeObject(c1);
				          oos.close();
				  
				          //创建对象出入流对象
				          ObjectInputStream ois = new ObjectInputStream(new FileInputStream("C:\\Users\\Think\\Desktop\\b.txt"));
				          //读取对象
				          Citation c2 = (Citation) ois.readObject();
				          //获取c2奖状所属学生对象
				          Student stu1 = c2.getStu();
				          stu1.setName("李四");
				  
				          //判断stu对象和stu1对象是否是同一个对象
				          System.out.println("stu和stu1是同一个对象？" + (stu == stu1));
				  
				          c1.show();
				          c2.show();
				      }
				  }
				  ```
				- ![原型模式扩展（深克隆）代码演示结果](https://www.panziye.com/wp-content/uploads/2022/06/202206030107353.png)
			- 通过上面的结果，我们可以看出，`stu`和`stu1`不是同一个对象，实现了深克隆。
	- ## 浅克隆(Java中的Object类中提供了 `clone()` 方法来实现浅克隆)
	  collapsed:: true
		- 创建一个新对象，新对象的属性和原来对象完全相同，对于非基本类型属性，仍指向原有属性所指向的对象的内存地址。
		- [[#red]]==**调用clone方法时并不会触发构造方法，而是触发实现的clone方法。**==
		- ## 例子1：
		  collapsed:: true
			- 具体原型类：必须实现Cloneable接口
				- ```java
				  public class Realizetype implements Cloneable {
				      // 构造函数
				      public Realizetype() {
				          System.out.println("具体的原型对象创建完成！");
				      }
				      
				      // 重写 clone方法
				      @Override
				      protected Realizetype clone() throws CloneNotSupportedException {
				          System.out.println("具体原型复制成功！");
				          return (Realizetype) super.clone();
				      }
				  }
				  ```
			- 测试访问类
				- ```java
				  public class PrototypeTest {
				      public static void main(String[] args) throws CloneNotSupportedException {
				          Realizetype r1 = new Realizetype();
				          Realizetype r2 = r1.clone();
				  
				          System.out.println("对象r1和r2是同一个对象？" + (r1 == r2));
				      }
				  }
				  ```
			- 输出结果是`false`，表明这个`r1`和`r2`不是同一个对象，并且只有在r1创建时`Realizetype`的构造方法执行了一次，调用`clone`方法时并不会触发构造方法，而是触发实现的`clone`方法。
		- ## 例子2：**用原型模式生成“三好学生”奖状**
		  collapsed:: true
			- 同一学校的“三好学生”奖状除了获奖人姓名不同，其他都相同，可以使用原型模式复制多个“三好学生”奖状出来，然后在修改奖状上的名字即可。
			  collapsed:: true
				- ```java
				  //奖状类
				  public class Citation implements Cloneable {
				      private String name;
				  
				      public void setName(String name) {
				          this.name = name;
				      }
				  
				      public String getName() {
				          return (this.name);
				      }
				  
				      public void show() {
				          System.out.println(name + "同学：在2020学年第一学期中表现优秀，被评为三好学生。特发此状！");
				      }
				  
				      @Override
				      public Citation clone() throws CloneNotSupportedException {
				          return (Citation) super.clone();
				      }
				  }
				  
				  //测试访问类
				  public class CitationTest {
				      public static void main(String[] args) throws CloneNotSupportedException {
				          Citation c1 = new Citation();
				          c1.setName("张三");
				  
				          //复制奖状
				          Citation c2 = c1.clone();
				          //将奖状的名字修改李四
				          c2.setName("李四");
				  
				          c1.show();
				          c2.show();
				      }
				  }
				  ```
			- 我们会发现，在调用`c1.show();`打印的学生是张三，调用`c2.show();`打印的学生是李四，成功实现了克隆后属性的修改。
- # 四、应用场景
  collapsed:: true
	- 对象的创建非常复杂，可以使用原型模式快捷的创建对象。
	- 性能和安全要求比较高。