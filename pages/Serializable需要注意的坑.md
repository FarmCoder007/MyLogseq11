# 一、多引用写入
	- ```java
	  public class Course implements Serializable {
	      private static final long serialVersionUID = 667279791530738499L;
	      private String name;
	      private float score;
	      ...
	      public static void main(String... args) throws Exception {
	          //TODO:
	          //TODO:
	          Course course = new Course("英语", 12f);
	          ByteArrayOutputStream out = new ByteArrayOutputStream();
	          ObjectOutputStream oos = new ObjectOutputStream(out);
	          oos.writeObject(course);
	          course.setScore(78f);
	          // oos.reset();
	          oos.writeUnshared(course);
	          // oos.writeObject(course);
	          byte[] bs = out.toByteArray();
	          oos.close();
	          ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(bs));
	          Course course1 = (Course) ois.readObject();
	          Course course2 = (Course) ois.readObject();
	          System.out.println("course1: " + course1);
	          System.out.println("course2: " + course2);
	      }
	  }
	  ```
	- 执行结果:
	- ```java
	  course1: Course{name='英语', score=12.0}
	  course2: Course{name='英语', score=12.0}
	  ```
	- 在默认情况下， 对于一个实例的多个引用，为了节省空间，只会写入一次，后面会追加几个字节代表某个实例的引用。
	- ## [[#red]]==一个对象，序列化一次，中间改值了，再写入需要调用reset 或者writeUnshared==。要不然同一个实例多引用只会写入一次
- # 二、子类实现序列化，父类不实现序列化/ 对象引用
	- ```java
	  public class Person {
	      private String name;
	      private String sax;
	      // public Person() {
	      // }
	      public Person(String name, String sax) {
	          this.name = name;
	          this.sax = sax;
	      }
	  }
	  public class Student1 extends Person implements Serializable {
	      private static final long serialVersionUID = -2100492893943893602L;
	      private Integer age;
	      private List<Course> courses;
	      public Student1(String name, String sax, Integer age) {
	          super(name,sax);
	          ...
	      }
	      ...
	      public static void main(String ... args) throws Exception{
	          //TODO:
	          Student1 student = new Student1("Zero", "男", 18);
	          student.addCourse(new Course("语文", 90.2f));
	          //序列化
	          byte[] bytes = SerializeableUtils.serialize(student);
	          System.out.println(Arrays.toString(bytes));
	          //反序列化
	          //在readObject时抛出java.io.NotSerializableException异常。
	          //需要Person添加一个无参数构造器
	          Student1 student1 = SerializeableUtils.deserialize(bytes);
	          System.out.println("Student: " + student1);
	      }
	  }
	  ```
	- 在readObject时抛出java.io.NotSerializableException异常。
- # 三、类的演化:反序列化目标类多一个字段(height)时
	- ```java
	  //反序列化目标类多一个字段(height)
	  public class Student implements Serializable {
	        private static final long serialVersionUID = -2100492893943893602L;
	        private String name;
	        private String sax;
	        private Integer age;
	        private List<Course> courses;
	        ...
	        @Override
	        public String toString() {
	            return "Student{" +
	            "name='" + name + '\'' +
	            ", sax='" + sax + '\'' +
	            ", age=" + age +
	            // ", height=" + height +
	            ", courses=" + courses +
	            '}';
	        }
	        //private float height;
	        public static void main(String ... args)throws Exception{
	              //TODO:
	              String path = System.getProperty("user.dir") +"/a.out";
	              // Student student = new Student("Zero", "男", 18);
	              // student.addCourse(new Course("语文", 90.2f));
	              // //序列化
	                // SerializeableUtils.saveObject(student,path);
	              //反序列化
	              Student student1 = SerializeableUtils.readObject(path);
	              System.out.println("Student: " + student1);
	        }
	  }
	  ```
	- 执行结果：
	- ```java
	  //序列化的时候
	  Student: Zero 男 18
	  Course: 语文 90.2
	  //反序列化的时候 添加一个float height
	  Student: Student{name='Zero', sax='男', age=18, height=0.0, courses=
	  [Course{name='语文', score=90.2}]}
	  ```
	- 可以看出反序列化之后，并没有报错，只是height实赋成了默认值。类似的其它对象也会赋值为默认值。
	- 还有 相反，如果写入的多一个字段，读出的少一个字段，也是不会报错的
	- 其它演化，比如更改类型等，这种演化本身就有问题，没必再探讨
- # 四、枚举类型（没问题，不看）
	- ```java
	  public enum Num {
	  	TWO, ONE, THREE;
	      public void printValues() {
	          System.out.println(ONE + " ONE.ordinal " + ONE.ordinal());
	          System.out.println(TWO + " TWO.ordinal " + TWO.ordinal());
	          System.out.println(THREE + " THREE.ordinal " + THREE.ordinal());
	      }
	      public static void testSerializable() throws Exception {
	          File file = new File("p.dat");
	          // ObjectOutputStream oos = new ObjectOutputStream(new
	          FileOutputStream(file));
	          // oos.writeObject(Num.ONE);
	          // oos.close();
	          Num.ONE.printValues();
	          System.out.println("=========反序列化后=======");
	          ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
	          Num s1 = (Num) ois.readObject();
	          s1.printValues();
	          ois.close();
	       }
	      public static void main(String... args) throws Exception {
	          //TODO:
	          testSerializable();
	      }
	  }
	  ```
	- 执行结果:
	- ```java
	  ONE ONE.ordinal 1
	  TWO TWO.ordinal 0
	  THREE THREE.ordinal 2
	  =========反序列化后=======
	  //调换(ONE,TWO)的位置: TWO, ONE, THREE; ->ONE, TWO, THREE;
	  ONE ONE.ordinal 0
	  TWO TWO.ordinal 1
	  THREE THREE.ordinal 2
	  ```
	- 可以看到ONE的值变成了0.
	- 事实上序列化Enum对象时，并不会保存元素的值，只会保存元素的name。这样，在不依赖元素值的
	- 前提下，ENUM对象如何更改都会保持兼容性。
	- kotlin 没事