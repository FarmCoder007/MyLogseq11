# 一、概念
	- AIDL（Android Interface Definition Language）是 [[#red]]==**Android 系统中用于跨进程通信（IPC）接口定义语言。**==它允许不同进程间的组件（如应用程序和系统服务）定义和共享接口，以便彼此之间进行通信和交互。
- # 作用
	- 使用Binder通信机制的时候，需要符合一些规则。AIDL就是帮我们遵循这些规则去实现通信。简化流程
	- 相当于黄牛等
- # 二、语法
  collapsed:: true
	- # 1、文件类型
		- 用AIDL书写的文件的后缀是 .aidl，而不是 .java。
	- # 2、数据类型：
		- AIDL默认支持一些数据类型，在使用这些数据类型的时候是不需要导包的，但是除了这些类型之外的数据类型，在使用之前必须导包，就算目标文件与当前正在编写的 .aidl 文件在同一个包下——在 Java 中，这种情况是不需要导包的。比如，现在我们编写了两个文件，一个叫做 Book.java ，另一个叫做 BookManager.aidl，它们都在 com.lypeer.aidldemo 包下 ，现在我们需要在 .aidl 文件里使用 Book 对象，那么我们就必须在 .aidl 文件里面写上 import com.lypeer.aidldemo.Book; 哪怕 .java 文件和 .aidl 文件就在一个包下。
		- ## 默认支持的数据类型包括：
			- Java中的八种基本数据类型，包括 byte，short，int，long，float，double，boolean，char。
			- String 类型。
			- CharSequence类型。
			- List类型：List中的所有元素必须是AIDL支持的类型之一，或者是一个其他AIDL生成的接口，或者是定义的parcelable（下文关于这个会有详解）。List可以使用泛型。
			- Map类型：Map中的所有元素必须是AIDL支持的类型之一，或者是一个其他AIDL生成的接口，或者是定义的parcelable。Map是不支持泛型的。
	- # 3、定向tag：（in，out,inout）
		- AIDL中的定向 tag 表示了在跨进程通信中数据的流向，
			- in 表示数据只能由客户端流向服务端
				- in 为定向 tag 的话表现为服务端将会接收到一个那个对象的完整数据，但是客户端的那个对象不会因为服务端对传参的修改而发生变动
			- out 表示数据只能由服务端流向客户端，
				- out 的话表现为服务端将会接收到那个对象的的空对象，但是在服务端对接收到的空对象有任何修改之后客户端将会同步变动；
			- inout 则表示数据可在服务端与客户端之间双向流通。
				- inout 为定向 tag 的情况下，服务端将会接收到客户端传来对象的完整信息，并且客户端将会同步服务端对该对象的任何变动。
		- 其中，==数据流向是针对在客户端中的那个传入方法的对象而言的
		- ## 参考
			- [你真的理解AIDL中的in，out，inout么？](https://blog.csdn.net/luoyanglizi/article/details/51958091)
		- ## 注意
			- Java 中的基本类型和 String ，CharSequence 的定向 tag 默认且只能是 in
			- 还有，请注意，请不要滥用定向 tag ，而是要根据需要选取合适的——要是不管三七二十一，全都一上来就用 inout ，等工程大了系统的开销就会大很多——因为排列整理参数的开销是很昂贵的。
	- # 4、两种AIDL文件：
		- ## 一类是用来定义parcelable对象，以供其他AIDL文件使用AIDL中非默认支持的数据类型的。
			- 注：所有的非默认支持数据类型必须通过第一类AIDL文件定义才能被使用。
			- *Book.aidl*
				- ```aidl
				  // Book.aidl
				  //第一类AIDL文件
				  //这个文件的作用是引入了一个序列化对象 Book 供其他的AIDL文件使用
				  //注意：Book.aidl与Book.java的包名应当是一样的
				  package com.lypeer.ipcclient;
				  
				  //注意parcelable是小写
				  parcelable Book;
				  ```
		- ## 一类是用来定义方法接口，以供系统使用来完成跨进程通信的。
			- ```aidl
			  // BookManager.aidl
			  //第二类AIDL文件
			  //作用是定义方法接口
			  package com.lypeer.ipcclient;
			  //导入所需要使用的非默认支持数据类型的包
			  import com.lypeer.ipcclient.Book;
			  
			  interface BookManager {
			  
			      //所有的返回值前都不需要加任何东西，不管是什么数据类型
			      List<Book> getBooks();
			  
			      //传参时除了Java基本类型以及String，CharSequence之外的类型
			      //都需要在前面加上定向tag，具体加什么量需而定
			      void addBook(in Book book);
			  }
			  ```
		- 可以看到，两类文件都是在“定义”些什么，而不涉及具体的实现，这就是为什么它叫做“Android接口定义语言”。
- # 二、[[AIDL和Binder的联系和区别]]
- # 三、[[AIDL基本使用]]
- # 四、[[AIDL原理]]
-
-
- # 四、坑
  collapsed:: true
	- ## 1、Book.aidl与Book.java的包名应当是一样的
		- 这里又有一个坑！大家可能注意到了，在 Book.aidl 文件中，我一直在强调：Book.aidl与Book.java的包名应当是一样的。这似乎理所当然的意味着这两个文件应当是在同一个包里面的——事实上，很多比较老的文章里就是这样说的，他们说最好都在 aidl 包里同一个包下，方便移植——然而在 Android Studio 里并不是这样。如果这样做的话，系统根本就找不到 Book.java 文件，从而在其他的AIDL文件里面使用 Book 对象的时候会报 Symbol not found 的错误。为什么会这样呢？因为 Gradle 。大家都知道，Android Studio 是默认使用 Gradle 来构建 Android 项目的，而 Gradle 在构建项目的时候会通过 sourceSets 来配置不同文件的访问路径，从而加快查找速度——问题就出在这里。Gradle 默认是将 java 代码的访问路径设置在 java 包下的，这样一来，如果 java 文件是放在 aidl 包下的话那么理所当然系统是找不到这个 java 文件的。那应该怎么办呢？
		- 又要 java文件和 aidl 文件的包名是一样的，又要能找到这个 java 文件——那么仔细想一下的话，其实解决方法是很显而易见的。首先我们可以把问题转化成：如何在保证两个文件包名一样的情况下，让系统能够找到我们的 java 文件？这样一来思路就很明确了：要么让系统来 aidl 包里面来找 java 文件，要么把 java 文件放到系统能找到的地方去，也即放到 java 包里面去。接下来我详细的讲一下这两种方式具体应该怎么做：
		- ## 解决方法一
			- 修改 build.gradle 文件：在 android{} 中间加上下面的内容
			  collapsed:: true
				- ```
				  sourceSets {
				      main {
				          java.srcDirs = ['src/main/java', 'src/main/aidl']
				      }
				  }
				  ```
		- ## 解决方法二
			- 把 java 文件放到 java 包下去：把 Book.java 放到 java 包里任意一个包下，保持其包名不变，与 Book.aidl 一致。只要它的包名不变，Book.aidl 就能找到 Book.java ，而只要 Book.java 在 java 包下，那么系统也是能找到它的。但是这样做的话也有一个问题，就是在移植相关 .aidl 文件和 .java 文件的时候没那么方便，不能直接把整个 aidl 文件夹拿过去完事儿了，还要单独将 .java 文件放到 java 文件夹里去。
- # 五、参考
	- [Android：学习AIDL，这一篇文章就够了(上)](https://blog.csdn.net/luoyanglizi/article/details/51980630)
	-
- # 六、[[AIDL面试题]]