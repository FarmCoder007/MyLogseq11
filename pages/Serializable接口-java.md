- # 一、背景
  collapsed:: true
	- 是 Java 提供的序列化接口，它是一个空接口：
		- ```java
		  public interface Serializable {
		  }
		  ```
	- Serializable 用来标识当前类可以被 ObjectOutputStream 序列化，以及被 ObjectInputStream 反序列化。
- # 二、Serializable入门使用
  collapsed:: true
	- ```java
	  public class Student implements Serializable {
	        //serialVersionUID唯一标识了一个可序列化的类
	        private static final long serialVersionUID = -2100492893943893602L;
	        private String name;
	        private String sax;
	        private Integer age;
	        //Course也需要实现Serializable接口
	        private List<Course> courses;
	        //用transient关键字标记的成员变量不参与序列化(在被反序列化后，transient 变量的值被
	        //设为初始值，如 int 型的是 0，对象型的是 null)
	        private transient Date createTime;
	        //静态成员变量属于类不属于对象，所以不会参与序列化(对象序列化保存的是对象的“状态”，也
	        // 就是它的成员变量，因此序列化不会关注静态变量)
	        private static SimpleDateFormat simpleDateFormat = new SimpleDateFormat();
	        public Student() {
	        		System.out.println("Student: empty");
	        }
	        public Student(String name, String sax, Integer age) {
	              System.out.println("Student: " + name + " " + sax + " " + age);
	              this.name = name;
	              this.sax = sax;
	              this.age = age;
	              courses = new ArrayList<>();
	              createTime = new Date();
	        }
	        ...
	        }
	        ////Course也需要实现Serializable接口
	        public class Course implements Serializable {
	              private static final long serialVersionUID = 667279791530738499L;
	              private String name;
	              private float score;
	        ...
	  }
	  ```
- # 三、Serializable 有以下几个细节点：
  collapsed:: true
	- 1、可序列化类中，未实现 Serializable 的成员变量 /属性 无法被序列化/反序列化
		- 待验证
		  collapsed:: true
			- 也就是说，反序列化一个类的过程中，它的非可序列化的属性将会调用无参构造函数重新创建
			- 因此这个属性的无参构造函数必须可以访问，否者运行时会报错
	- 2、一个实现序列化的类，它的子类也是可序列化的
- # 四、serialVersionUID与兼容性
  collapsed:: true
	- serialVersionUID的作用
		- serialVersionUID 用来表明[[#red]]==**类的不同版本间的兼容性**==。如果你修改了此类, 要修改此值。否则以前用老版本的类序列化的类恢复时会报错: InvalidClassException
	- 设置方式
		- 在JDK中，可以利用JDK的bin目录下的serialver.exe工具产生这个serialVersionUID，对于Test.class，执行命令：serialver Test
	- 兼容性问题
		- 为了在反序列化时，确保类版本的兼容性，最好在每个要序列化的类中加入 private static final long serialVersionUID这个属性，具体数值自己定义。这样，即使某个类在与之对应的对象 已经序列化出去后做了修改，该对象依然可以被正确反序列化。否则，如果不显式定义该属性，这个属性值将由JVM根据类的相关信息计算，而修改后的类的计算 结果与修改前的类的计算结果往往不同，从而造成对象的反序列化因为类版本不兼容而失败。
		- 不显式定义这个属性值的另一个坏处是，不利于程序在不同的JVM之间的移植。因为不同的编译器实现该属性值的计算策略可能不同，从而造成虽然类没有改变，但是因为JVM不同，出现因类版本不兼容而无法正确反序列化的现象出现
	- 因此 JVM 规范强烈 建议我们手动声明一个版本号，这个数字可以是随机的，只要固定不变就可以。同时最好是 private 和 final 的，尽量保证不变。
- # 五、子类Externalizable接口：可以自己决定哪些序列化
  collapsed:: true
	- 接口
		- ```java
		  public interface Externalizable extends Serializable {
		        /*
		        * 怎么把对象写入到 var1
		        */
		        void writeExternal(ObjectOutput var1) throws IOException;
		        /**
		        * 怎么把对象从 var1读取出来
		        */
		        void readExternal(ObjectInput var1) throws IOException,ClassNotFoundException;
		  }
		  ```
	- 简单使用
		- ```java
		  public class Course1 implements Externalizable {
		          private static final long serialVersionUID = 667279791530738499L;
		          private String name;
		          private float score;
		          ...
		          @Override
		          public void writeExternal(ObjectOutput objectOutput) throws IOException
		          {
		                System.out.println("writeExternal");
		                objectOutput.writeObject(name);
		                objectOutput.writeFloat(score);
		          }
		          @Override
		          public void readExternal(ObjectInput objectInput) throws IOException,
		          													ClassNotFoundException {
		                System.out.println("readExternal");
		                name = (String)objectInput.readObject();
		                score = objectInput.readFloat();
		          }
		          ...
		          public static void main(String... args) throws Exception {
		                //TODO:
		                Course1 course = new Course1("英语", 12f);
		                // 序列化到二进制流
		                ByteArrayOutputStream out = new ByteArrayOutputStream();
		                ObjectOutputStream oos = new ObjectOutputStream(out);
		                oos.writeObject(course);
		                course.setScore(78f);
		                byte[] bs = out.toByteAray()    
		                // 以上序列化完成
		            
		                oos.close();
		                // 从二进制流反序列化
		                ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(bs));
		                Course1 course1 = (Course1) ois.readObject();
		                System.out.println("course1: " + course1);
		          }
		  ```
- # 六、[[序列化与反序列化Serializable]]
- # 七、Java的序列化步骤与数据结构分析
  collapsed:: true
	- ## 序列化算法一般会按步骤做如下事情：
		- 将对象实例相关的类元数据输出。
		- 递归地输出类的超类描述直到不再有超类。
		- 类元数据完了以后，开始从最顶层的超类开始输出对象实例的实际数据值。
		- 从上至下递归输出实例的数据
	- ## 格式化后以二进制打开
		- ```java
		  aced 0005 7372 002e 636f 6d2e 7a65 726f
		  2e73 6572 6961 6c69 7a61 626c 6564 656d
		  6f2e 7365 7269 616c 697a 6162 6c65 2e53
		  7475 6465 6e74 e2d9 8cd7 833d f19e 0200
		  044c 0003 6167 6574 0013 4c6a 6176 612f
		  6c61 6e67 2f49 6e74 6567 6572 3b4c 0007
		  636f 7572 7365 7374 0010 4c6a 6176 612f
		  7574 696c 2f4c 6973 743b 4c00 046e 616d
		  6574 0012 4c6a 6176 612f 6c61 6e67 2f53
		  7472 696e 673b 4c00 0373 6178 7100 7e00
		  0378 7073 7200 116a 6176 612e 6c61 6e67
		  2e49 6e74 6567 6572 12e2 a0a4 f781 8738
		  0200 0149 0005 7661 6c75 6578 7200 106a
		  6176 612e 6c61 6e67 2e4e 756d 6265 7286
		  ac95 1d0b 94e0 8b02 0000 7870 0000 0012
		  7372 0013 6a61 7661 2e75 7469 6c2e 4172
		  7261 794c 6973 7478 81d2 1d99 c761 9d03
		  0001 4900 0473 697a 6578 7000 0000 0277
		  0400 0000 0273 7200 2d63 6f6d 2e7a 6572
		    6f2e 7365 7269 616c 697a 6162 6c65 6465
		  6d6f 2e73 6572 6961 6c69 7a61 626c 652e
		  436f 7572 7365 0942 a76f 5bfc 8343 0200
		  0246 0005 7363 6f72 654c 0004 6e61 6d65
		  7100 7e00 0378 7042 b466 6674 0006 e8af
		  ade6 9687 7371 007e 000a 42b2 999a 7400
		  06e6 95b0 e5ad a678 7400 045a 6572 6f74
		  0003 e794 b7
		  ```
		- AC ED: STREAM_MAGIC. 声明使用了序列化协议.
		- 00 05: STREAM_VERSION. 序列化协议版本.
		- 0x73: TC_OBJECT. 声明这是一个新的对象.
		- 0x72: TC_CLASSDESC. 声明这里开始一个新Class。
		- 00 2e: Class名字的长度.
- # 八、[[readObject/writeObject原理分析]]
- # 九、[[Serializable需要注意的坑]]
- # 十、序列化虽说不需要重写方法，但是有这四个，你写了他会反射调用。重写readObject,writeObject
  collapsed:: true
	- > “只有当你自行设计的自定义序列化形式与默认的序列化形式基本相同时，才能接受默认的序列化形式”.“当一个对象的物理表示方法与它的逻辑数据内容有实质性差别时，使用默认序列化形式有 N种缺陷”.其实从effffective java的角度来讲，是强烈建议我们重写的，这样有助于我们更好地把控序列化过程，防范未知风险
	- ```java
	  public class Course3 implements Serializable {
	        private static final long serialVersionUID = 667279791530738499L;
	        private String name;
	        private float score;
	        ...
	        private void readObject(ObjectInputStream inputStream) throws ClassNotFoundException, 
	    																		IOException {
	              System.out.println("readObject");
	              inputStream.defaultReadObject();
	              name = (String)inputStream.readObject();
	              score = inputStream.readFloat();
	        }
	        private void writeObject(ObjectOutputStream outputStream) throws IOException {
	              System.out.println("writeObject");
	              outputStream.defaultWriteObject();
	              outputStream.writeObject(name);
	              outputStream.writeFloat(score);
	        }
	        private Object readResolve() {
	              System.out.println("readResolve");
	              return new Course3(name, 85f);
	        }
	    
	        /**
	         
	        */
	        private Object writeReplace(){
	              System.out.println("writeReplace");
	              return new Course3(name +"replace",score);
	        }
	  }
	    ...
	  public static void main(String... args) throws Exception {
	      //TODO:
	      Course3 course = new Course3("英语", 12f);
	      ByteArrayOutputStream out = new ByteArrayOutputStream();
	      ObjectOutputStream oos = new ObjectOutputStream(out);
	      oos.writeObject(course);
	      byte[] bs = out.toByteArray();
	      oos.close();
	      ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(bs));
	      Course3 course1 = (Course3) ois.readObject();
	      System.out.println("course1: " + course1);
	      }
	  }
	  ```
	- 执行结果：
	- ```java
	  Course: 英语 12.0
	  writeReplace
	  Course: 英语replace 12.0
	  writeObject
	  readObject
	  readResolve
	  Course: 英语replace 85.0
	  course1: Course{name='英语replace', score=85.0}
	  ```
	- writeReplace 先于writeObject
	- readResolve后于readObject
- # 十一、单例模式的序列化问题/反射问题
  collapsed:: true
	- ```java
	  public class SingleTest {
	  	static final String CurPath = System.getProperty("user.dir");
	      public static void main(String ... args) throws Exception {
	          //TODO:
	          Single instance = Single.getInstance();
	          System.out.println(instance.hashCode());
	          System.out.println(copyInstance(instance).hashCode());
	          System.out.println("=================反射======================");
	          //使用反射方式直接调用私有构造器
	          Class<Single> clazz = (Class<Single>)Class.forName("com.zero.serializabledemo.serializable.Single");
	          Constructor<Single> con = clazz.getDeclaredConstructor(null);
	          con.setAccessible(true);//绕过权限管理，即在true的情况下，可以通过构造函数新建对象
	          Single instance1 = con.newInstance();
	          Single instance2 = con.newInstance();
	          System.out.println(instance1.hashCode());
	          System.out.println(instance2.hashCode());
	      }
	      private static Single copyInstance(Single instance) throws Exception{
	          //序列化会导致单例失效
	          FileOutputStream fos = new FileOutputStream(CurPath+"/a.txt");
	          ObjectOutputStream oos = new ObjectOutputStream(fos);
	          oos.writeObject(instance);
	          ObjectInputStream ois = new ObjectInputStream(new
	          FileInputStream(CurPath+"/a.txt"));
	          Single single2 = (Single)ois.readObject();
	          oos.close();
	          ois.close();
	          return single2;
	      }
	  }
	  class Single implements Serializable {
	      private static final long serialVersionUID = 1L;
	      private static boolean flag = false;
	      private Single(){
	          synchronized (Single.class) {
	              if (!flag) {
	                  // flag = true;
	              } else {
	                  throw new RuntimeException("单例模式被侵犯！");
	              }
	          }
	      }
	      private static Single single;
	      public static Single getInstance(){
	          if (single == null ) {
	              synchronized (Single.class) {
	                  if ( single == null ) {
	                  	single = new Single();
	                  }
	              }
	          }
	          return single;
	      }
	      //如果不重写readResolve,会导致单例模式在序列化->反序列化后失败
	      // private Object readResolve() {
	      // return single;
	      // }
	  }
	  ```