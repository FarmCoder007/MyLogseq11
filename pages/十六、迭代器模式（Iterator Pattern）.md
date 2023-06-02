- # 一、迭代器模式定义
	- 提供一个对象来顺序访问聚合对象中的一系列数据，而不暴露聚合对象的内部表示。
- # 二、结构
	- 迭代器模式主要包含以下角色：
	- **抽象聚合（Aggregate）角色：**定义存储、添加、删除聚合元素以及创建迭代器对象的接口。
	- **具体聚合（ConcreteAggregate）角色：**实现抽象聚合类，返回一个具体迭代器的实例。
	- **抽象迭代器（Iterator）角色：**定义访问和遍历聚合元素的接口，通常包含 hasNext()、next() 等方法。
	- **具体迭代器（Concretelterator）角色：**实现抽象迭代器接口中所定义的方法，完成对聚合对象的遍历，记录遍历的当前位置。
- # 三、案例实现
  collapsed:: true
	- 定义一个可以存储学生对象的容器对象，将遍历该容器的功能交由迭代器实现，涉及到的类如下：
	  collapsed:: true
		- ![迭代器模式类图](https://www.panziye.com/wp-content/uploads/2022/06/2022061402435547.png)
	- ### 学生类
	  collapsed:: true
		- ```java
		  public class Student {
		      private String name;
		      private String number;
		   
		      @Override
		      public String toString() {
		          return "Student{" +
		                  "name='" + name + '\'' +
		                  ", number='" + number + '\'' +
		                  '}';
		      }
		   
		      public String getName() {
		          return name;
		      }
		   
		      public void setName(String name) {
		          this.name = name;
		      }
		   
		      public String getNumber() {
		          return number;
		      }
		   
		      public void setNumber(String number) {
		          this.number = number;
		      }
		   
		      public Student(String name, String number) {
		          this.name = name;
		          this.number = number;
		      }
		   
		      public Student() {
		      }
		  }
		  ```
	- ### 定义迭代器接口，声明hasNext、next方法：
	  collapsed:: true
		- ```java
		  // 抽象迭代器角色接口
		  public interface StudentIterator {
		   
		      //判断是否还有元素
		      boolean hasNext();
		   
		      //获取下一个元素
		      Student next();
		  }
		  ```
	- ### 具体的迭代器类，重写所有的抽象方法:
	  collapsed:: true
		- ```java
		  //具体迭代器角色类
		  public class StudentIteratorImpl implements StudentIterator {
		   
		      private List<Student> list;
		      private int position = 0;//用来记录遍历时的位置
		   
		      public StudentIteratorImpl(List<Student> list) {
		          this.list = list;
		      }
		   
		      public boolean hasNext() {
		          return position < list.size();
		      }
		   
		      public Student next() {
		          //从集合中获取指定位置的元素
		          Student currentStudent = list.get(position);
		          position++;
		          return currentStudent;
		      }
		  }
		  ```
	- ### 抽象容器类，包含添加元素，删除元素，获取迭代器对象的方法:
	  collapsed:: true
		- ```java
		  //抽象聚合角色接口
		  public interface StudentAggregate {
		   
		      //添加学生功能
		      void addStudent(Student stu);
		   
		      //删除学生功能
		      void removeStudent(Student stu);
		   
		      //获取迭代器对象功能
		      StudentIterator getStudentIterator();
		  }
		  ```
	- ### 定义具体的容器类，重写所有的方法:
	  collapsed:: true
		- ```java
		  //具体聚合角色
		  public class StudentAggregateImpl implements StudentAggregate {
		   
		      private List<Student> list = new ArrayList<Student>();
		   
		      public void addStudent(Student stu) {
		          list.add(stu);
		      }
		   
		      public void removeStudent(Student stu) {
		          list.remove(stu);
		      }
		   
		      //获取迭代器对象
		      public StudentIterator getStudentIterator() {
		          return new StudentIteratorImpl(list);
		      }
		  }
		  ```
	- ### 测试类：
	  collapsed:: true
		- ```java
		  public class Client {
		      public static void main(String[] args) {
		          //创建聚合对象
		          StudentAggregateImpl aggregate = new StudentAggregateImpl();
		          //添加元素
		          aggregate.addStudent(new Student("张三","001"));
		          aggregate.addStudent(new Student("李四","002"));
		          aggregate.addStudent(new Student("王五","003"));
		          aggregate.addStudent(new Student("赵六","004"));
		   
		          //遍历聚合对象
		   
		          //1,获取迭代器对象
		          StudentIterator iterator = aggregate.getStudentIterator();
		          //2,遍历
		          while(iterator.hasNext()) {
		              //3,获取元素
		              Student student = iterator.next();
		              System.out.println(student.toString());
		          }
		      }
		  }
		  ```
	- 运行效果：
		- ![运行测试结果](https://www.panziye.com/wp-content/uploads/2022/06/202206140251508.png)
- # 四、迭代器模式优缺点
	- ### 1，优点：
		- 它支持以不同的方式遍历一个聚合对象，在同一个聚合对象上可以定义多种遍历方式。在迭代器模式中只需要用一个不同的迭代器来替换原有迭代器即可改变遍历算法，我们也可以自己定义迭代器的子类以支持新的遍历方式。
		- 迭代器简化了聚合类。由于引入了迭代器，在原有的聚合对象中不需要再自行提供数据遍历等方法，这样可以简化聚合类的设计。
		- 在迭代器模式中，由于引入了抽象层，增加新的聚合类和迭代器类都很方便，无须修改原有代码，满足 “开闭原则” 的要求。
	- ### 2，缺点：
		- 增加了类的个数，这在一定程度上增加了系统的复杂性。
- # 五、使用场景
	- 当需要为聚合对象提供多种遍历方式时。
	- 当需要为遍历不同的聚合结构提供一个统一的接口时。
	- 当访问一个聚合对象的内容而无须暴露其内部细节的表示时。
- # 六、JDK源码应用迭代器模式解析
	- 迭代器模式在JAVA的很多集合类中被广泛应用，接下来看看JAVA源码中是如何使用迭代器模式的。
	- ```java
	  List<String> list = new ArrayList<>();
	  Iterator<String> iterator = list.iterator(); //list.iterator()方法返回的肯定是Iterator接口的子实现类对象
	  while (iterator.hasNext()) {
	      System.out.println(iterator.next());
	  }
	  ```
	- 看完这段代码是不是很熟悉，与我们上面代码基本类似。单列集合都使用到了迭代器，我们以ArrayList举例来说明
		- List：抽象聚合类
		- ArrayList：具体的聚合类
		- Iterator：抽象迭代器
		- list.iterator()：返回的是实现了 Iterator 接口的具体迭代器对象
	- 具体的来看看 ArrayList的代码实现
	  collapsed:: true
		- ```java
		  public class ArrayList<E> extends AbstractList<E>
		          implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
		      
		      public Iterator<E> iterator() {
		          return new Itr();
		      }
		      
		      private class Itr implements Iterator<E> {
		          int cursor;       // 下一个要返回元素的索引
		          int lastRet = -1; // 上一个返回元素的索引
		          int expectedModCount = modCount;
		  ​
		          Itr() {}
		          
		          //判断是否还有元素
		          public boolean hasNext() {
		              return cursor != size;
		          }
		  ​
		          //获取下一个元素
		          public E next() {
		              checkForComodification();
		              int i = cursor;
		              if (i >= size)
		                  throw new NoSuchElementException();
		              Object[] elementData = ArrayList.this.elementData;
		              if (i >= elementData.length)
		                  throw new ConcurrentModificationException();
		              cursor = i + 1;
		              return (E) elementData[lastRet = i];
		          }
		          ...
		  }
		  ```
	- 这部分代码还是比较简单，大致就是在 iterator 方法中返回了一个实例化的 Iterator 对象。Itr是一个内部类，它实现了 Iterator 接口并重写了其中的抽象方法。
- # 七、实际使用
	- >当我们在使用JAVA开发的时候，想使用迭代器模式的话，只要让我们自己定义的容器类实现java.util.Iterable并实现其中的iterator()方法使其返回一个 java.util.Iterator 的实现类就可以了。