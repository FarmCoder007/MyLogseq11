# 1、AIDL作用
	- 使用Binder通信机制的时候，需要符合一些规则。AIDL就是帮我们遵循这些规则去实现通信。简化流程
- # 2、定向tag：（in，out,inout）
	- 定向 tag 表示[[#red]]==**跨进程通信中数据的流向**==，数据流向[[#red]]==**是针对在客户端中的调用AIDL中方法入参的那个对象**==而言的
		- ==**in**== 表示数据只能由==**客户端流向服务端**==
			- in 为定向 tag 的话表现为==**服务端将会接收到一个那个对象的完整数据**==，该对象服务端修改客户端不会得到同步
		- ==**out**== 表示数据只能由==**服务端流向客户端**==，
			- out 的话表现为==**服务端将会接收到那个对象的的空对象**==，该对象服务端修改客户端会得到同步
		- [[#red]]==**inout 则表示数据可在服务端与客户端之间双向流通。**==
			- inout 为定向 tag 的情况下，服务端将会接收到客户端传来对象的完整信息，并且客户端将会同步服务端对该对象的任何变动。
	- ## 注意
		- Java 中的基本类型和 String ，CharSequence 的定向 tag 默认且只能是 in
		- 还有，请注意，请不要滥用定向 tag ，而是要根据需要选取合适的——要是不管三七二十一，全都一上来就用 inout ，等工程大了系统的开销就会大很多——因为排列整理参数的开销是很昂贵的。
- # 3、[[AIDL和Binder的联系和区别]]
	- AIDL 和 Binder 是 Android 系统中的两个关键组件，共同用于实现进程间通信。==**AIDL 定义了接口规范，**==而 [[#red]]==**Binder 提供了底层的通信机制**==，使得不同进程间的组件能够进行交互和通信。
- # 4、[[AIDL基本使用]]
	- # 1、服务端
		- 1、创建IPersonManager==**.aidl文件定义了暴露服务端的方法**==。
		- 2、build后会生成对应IPersonManager.java文件里边==**包含IPersonManager.Stub类是个IBinder子类**==。==**相当于Binder对象**==。还有[[#red]]==**Stub.Proxy  Binder的代理对象**==
		- 3、[[#green]]==**声明服务service，**==继承Service，重写onBind() 函数，新建类继承IPersonManager.Stub，，作为IBinder在onBind()函数返回。客户端会通过ServiceConnection接收到的
		- 4、清单文件注册服务
	- # 2、客户端
		- 1、bindService 启动服务，传入ServiceConnection.
		- 2、在onServiceConnected 方法中 可以拿到服务端onBind()方法返回的IBinder对象。可通过IPersonManager.Stub.asInterface(iBinder)，将Ibinder转换为IPersonManager接口。
		- 其实就是 拿到服务端 binder的代理对象[[#green]]==**Stub.proxy**==。可通过代理对象调用服务端暴雷的方法的
- # 1、讲解AIDL生成代码详细情况讲解[[AIDL原理]]
	- ## 1、Stub.asInterface
		- 1、==**判断是否和客户端同进程**==，如果==**同进程有接口实现类**== 直接返回
		- 2、跨进程，返回IPersonManager的[[#red]]==**代理类Stub.Proxy**==，==**持有Stub这个继承Binder的类。给客户端用，实现跨进程**==
			- 解释：Stub继承了Binder类能底层跨进程通信，外边包一层Stub.Proxy是为了调用定义的接口方法
	- ## 2、客户端拿到代理对象Stub.Proxy，调用addPerson方法，怎么实现跨进程通信的？
		- 客户端用代理对象，调用远程服务的接口方法，addPerson，传入person对象
			- ```java
			  iPersonManager.addPerson(new Personon("xxx", 3));
			  ```
		- 看具体代理类Proxy的实现，怎么实现跨进程，transac
			- ```java
			      @Override
			      public void addPerson(Person person) throws RemoteException {
			          // 客户端 给 服务端的数据 打包到data
			          Parcel data = Parcel.obtain();
			          // 服务端 返回 给 客户端的数据 打包到 reply
			          Parcel reply = Parcel.obtain();
			          try {
			              data.writeInterfaceToken(DESCRIPTOR);
			              if ((person != null)) {
			                  data.writeInt(1);
			                  person.writeToParcel(data, 0);
			              } else {
			                  data.writeInt(0);
			              }
			              Log.e("leo", "Proxy,addPerson: " + Thread.currentThread());
			              // 调用  IBinder的 transact 实现跨进程 进行远程方法调用 
			              mRemote.transact(Stub.TRANSACTION_addPerson, data, reply, 0);
			              reply.readException();
			          } finally {
			              reply.recycle();
			              data.recycle();
			          }
			      }
			  ```
		- ##  [[#red]]==代理类中方法调用 addPerson做的事情==
			- 1、将 客户端数据 和 服务端数据[[#red]]==**打包**==
			- 2、然后调用[[#red]]==**Ibinder.transact()**==，传入命令TRANSACTION_addPerson和打包数据，告诉服务端调用的哪个方法执行跨进程
				- （同步的情况，客户端调用后，当前线程会挂起，等服务器返回数据）
				- 异步情况，不会挂起
	- ## 3、2中是调用的IBinder的transact方法解析[[解析3、transact方法]]
		- ```java
		  mRemote.transact(Stub.TRANSACTION_addPerson, _data, _reply, 0);
		  ```
		- 1、参数解读
			- 第一个参数：Stub中的常量Stub.TRANSACTION_addPerson。代表客户端调用服务端的哪个方法。
			- Stub服务端binder收到请求后来根据方法标号去匹配方法，去具体处理
		- 2、==**客户端调用transact方法**==，一般都是异步请求，请求后会被挂起
		- 3、然后进入[[#red]]==**服务端的Stub的onTransact()方法**==，根据传入的方法标号匹配，处理客户端的具体请求
		- 4、看对应方法标号分支，调用了[[#green]]==**this.addPerson(arg0)。其实就是Stub的子类**==。我们在创建远程Service时。OnBind()函数 返回的那个对象。对应的方法、
		- 5、由此 客户端和 服务端实现了通信