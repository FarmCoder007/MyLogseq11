- ==获取Service Manager是通过defaultServiceManager()方法来完成。==
- # 1.defaultServiceManager
	- frameworks/native/libs/binder/IServiceManager.cpp
	- ```c
	  // 33
	  sp<IServiceManager> defaultServiceManager()
	  {
	        /* 单例模式，如果不为空直接返回 */
	        if (gDefaultServiceManager != NULL) return gDefaultServiceManager;
	        {
	            AutoMutex _l(gDefaultServiceManagerLock);
	            /* 创建或者获取 SM时，SM可能未准备就绪，这时通过 sleep 1秒后，循环尝试获取直到成功
	            */
	            while (gDefaultServiceManager == NULL) {
	                /* 分为三块分析 */
	                gDefaultServiceManager = interface_cast<IServiceManager>(
	                									ProcessState::self()->getContextObject(NULL));
	                if (gDefaultServiceManager == NULL)
	                		sleep(1);
	            }
	         }
	  	return gDefaultServiceManager;
	  }
	  ```
	- ## 1-1.ProcessState::self： 实例化 ProcessState
	  collapsed:: true
		- ```c
		  frameworks/native/libs/binder/ProcessState.cpp
		  // 70
		  sp<ProcessState> ProcessState::self()
		  {
		        /* 单例模式 */
		        if (gProcess != NULL) {
		        		return gProcess;
		        }
		        gProcess = new ProcessState; // 实例化 ProcessState
		        return gProcess;
		  }
		  ```
		- ## 1-1-1.ProcessState::ProcessState
			- ```c
			  frameworks/native/libs/binder/ProcessState.cpp
			  // 339
			  ProcessState::ProcessState()
			  	: mDriverFD(open_driver())
			        
			  // 358 采用内存映射函数 mmap，给 binder分配一块大小为 (1M-8K)的虚拟地址空间,用来接收事务
			  mVMStart = mmap(0, BINDER_VM_SIZE, PROT_READ, MAP_PRIVATE | MAP_NORESERVE,
			  mDriverFD, 0);
			  ```
			- ### 1-1-1-1.open_driver
			  collapsed:: true
				- ```c
				  frameworks/native/libs/binder/ProcessState.cpp
				  // 311
				  static int open_driver()
				  // 313 打开 /dev/binder设备，建立与内核的 Binder驱动的交互通道
				  int fd = open("/dev/binder", O_RDWR);
				  // 328 通过 ioctl设置 binder驱动，能支持的最大线程数
				  size_t maxThreads = DEFAULT_MAX_BINDER_THREADS;
				  result = ioctl(fd, BINDER_SET_MAX_THREADS, &maxThreads);
				  ```
		- # 总结
			- 1、打开 /dev/binder设备，建立与内核的 Binder驱动的交互通道
			- 2、通过 ioctl设置 binder驱动，1个进程有一个Binder线程池，能支持的最大线程数  15个
			- 3、用mmap，给 binder驱动和服务端Service （普通服务,） 的共享内存大小 ： (1M-8K)的虚拟地址空间,用来接收事务
				- 这个普通服务是用来 获取SM用的
	- ## 1-2.ProcessState::getContextObject
	  collapsed:: true
		- ```c
		  frameworks/native/libs/binder/ProcessState.cpp
		  // 85
		  sp<IBinder> ProcessState::getContextObject(const sp<IBinder>& /*caller*/)
		  {
		      // 参数为0，获取service_manager服务  SM 的Handle  = 0
		      return getStrongProxyForHandle(0);
		  }
		  ```
		- ### 1-2-1.ProcessState::getStrongProxyForHandle
		  collapsed:: true
			- ```c
			  frameworks/native/libs/binder/ProcessState.cpp
			  // 179
			  sp<IBinder> ProcessState::getStrongProxyForHandle(int32_t handle)
			  {
			        // 查找 handle对应的资源项
			        handle_entry* e = lookupHandleLocked(handle);
			    
			        // 192 当handle值所对应的IBinder不存在或弱引用无效时
			        if (b == NULL || !e->refs->attemptIncWeak(this)) {
			              // 214 通过ping操作测试binder是否准备就绪
			              status_t status = IPCThreadState::self()->transact(
			                      0, IBinder::PING_TRANSACTION, data, NULL, 0);
			        }  
			        // 220 创建BpBinder对象
			        b = new BpBinder(handle);
			  }
			  ```
		- ### 1-2-2.BpBinder::[[BpBinder]]
		  collapsed:: true
			- ```c
			  frameworks/native/libs/binder/BpBinder.cpp
			  // 89
			  BpBinder::BpBinder(int32_t handle): mHandle(handle)
			    {
			      /* 支持强弱引用计数，OBJECT_LIFETIME_WEAK表示目标对象的生命周期受弱指针控制 */
			      extendObjectLifetime(OBJECT_LIFETIME_WEAK);
			      /* handle所对应的 bindle弱引用 + 1 */
			      IPCThreadState::self()->incWeakHandle(handle);
			    }
			  ```
		- # 总结
			- 1、创建一个BpBinder(客户端对象)SM的代理对象 [[Native层怎么获取BpBinder的]]
	- ## 1-3.interface_cast,相当于传入了BpBinder对象
	  id:: 64a22081-7704-4ccb-b85a-7f11a85e73a9
		- ```c
		  frameworks/native/include/binder/IInterface.h
		  // 41
		  template<typename INTERFACE>
		  inline sp<INTERFACE> interface_cast(const sp<IBinder>& obj)
		  {
		      // 等价于：IServiceManager::asInterface   相当于C的泛型
		      return INTERFACE::asInterface(obj);
		  }
		  ```
		- ### 1-3-1.IServiceManager::asInterface
		  collapsed:: true
			- 对于asInterface()函数，通过搜索代码，你会发现根本找不到这个方法是在哪里定义这个函数的, 其实是
			- 通过模板函数来定义的。
			- ```c
			  frameworks/native/include/binder/IInterface.h
			  // 74
			  #define DECLARE_META_INTERFACE(INTERFACE) 
			      static const android::String16 descriptor; 
			      static android::sp<I##INTERFACE> asInterface( 
			     		 const android::sp<android::IBinder>& obj); 
			      virtual const android::String16& getInterfaceDescriptor() const; 
			      I##INTERFACE(); 
			      virtual ~I##INTERFACE(); 
			  
			  // 83 相当于java的实现
			  #define IMPLEMENT_META_INTERFACE(INTERFACE, NAME) 
			      const android::String16 I##INTERFACE::descriptor(NAME); 
			      const android::String16& 
			      	I##INTERFACE::getInterfaceDescriptor() const { 
			      	return I##INTERFACE::descriptor; 
			      } 
			      android::sp<I##INTERFACE> I##INTERFACE::asInterface( 
			      	const android::sp<android::IBinder>& obj) 
			  { 
			    android::sp<I##INTERFACE> intr; 
			    if (obj != NULL) { 
			    		intr = static_cast<I##INTERFACE*>( 
			    			obj->queryLocalInterface( 
			    				I##INTERFACE::descriptor).get()); 
			          if (intr == NULL) { 
			          	intr = new Bp##INTERFACE(obj); 
			          } 
			    	} 
			    	return intr; 
			    } 
			    I##INTERFACE::I##INTERFACE() { } 
			    I##INTERFACE::~I##INTERFACE() { }
			  ```
		- ### 1-3-2.DECLARE_META_INTERFACE 相当于java接口的声明
			- ```c
			  frameworks/native/include/binder/IServiceManager.h
			  // 33
			  DECLARE_META_INTERFACE(ServiceManager)
			  ```
			- 展开即可得：
			- ```c
			  static const android::String16 descriptor;
			  static android::sp< IServiceManager > asInterface(const
			  android::sp<android::IBinder>& obj)
			  virtual const android::String16& getInterfaceDescriptor() const;
			  IServiceManager ();
			  virtual ~IServiceManager();
			  ```
			- 该过程主要是声明asInterface(),getInterfaceDescriptor()方法。
		- ### 1-3-3.IMPLEMENT_META_INTERFACE
			- ```c
			  frameworks/native/libs/binder/IServiceManager.cpp
			  // 185
			  IMPLEMENT_META_INTERFACE(ServiceManager,"android.os.IServiceManager")
			  ```
			- 展开即可得：
			- ```c
			  const android::String16
			  IServiceManager::descriptor(“android.os.IServiceManager”);
			  const android::String16& IServiceManager::getInterfaceDescriptor() const
			  {
			  		return IServiceManager::descriptor;
			  }
			  android::sp<IServiceManager> IServiceManager::asInterface(const
			  android::sp<android::IBinder>& obj)
			  {
			        android::sp<IServiceManager> intr;
			        if(obj != NULL) {
			        		intr = static_cast<IServiceManager *>(
			        		obj->queryLocalInterface(IServiceManager::descriptor).get());
			        		if (intr == NULL) {
			                  // 等价于 new BpServiceManager(BpBinder)
			                  intr = new BpServiceManager(obj);
			        	}
			        }
			        return intr;
			  }
			  IServiceManager::IServiceManager () { }
			  IServiceManager::~ IServiceManager() { }
			  ```
	- ## 1-4.BpServiceManager
		- ```c
		  frameworks/native/libs/binder/IServiceManager.cpp
		  // 126
		  class BpServiceManager : public BpInterface<IServiceManager>
		  // 129  impl 这个参数 还是开始传入的BpBinder
		  BpServiceManager(const sp<IBinder>& impl): BpInterface<IServiceManager>(impl)
		  ```
		- ### 1-4-1.BpInterface::BpInterface
			- ```c
			  frameworks/native/include/binder/IInterface.h
			  // 134
			  template<typename INTERFACE>
			  inline BpInterface<INTERFACE>::BpInterface(const sp<IBinder>& remote)
			  : BpRefBase(remote)
			  ```
		- ### 1-4-2.BpRefBase
			- ```c
			  frameworks/native/libs/binder/Binder.cpp
			  // 241 mRemote指向 new BpBinder(0)，从而 BpServiceManager能够利用 Binder进行通过通信
			  BpRefBase::BpRefBase(const sp<IBinder>& o)
			  : mRemote(o.get()), mRefs(NULL), mState(0)
			  ```
		-
	- # 1.3和1.4总结（Native层的流程）
		- 1、asInterface（BpBinder）相当于 new BpServiceManager（new BpBinder） 创建BpServiceManager传入一个BpBinder
		- 2、remote.transact (remote就是BpBinder ，通过BpBinder  和binder内核  进行远程调用)
	- # java层大概流程
		- 1、new ServiceManagerProxy(new BinderProxy)
		- 2、mRomote = BinderProxy
		- 2、BinderProxy.mObject = BpBinder
		- 4、mRomote.transact 就相当于BpBinder.transact
		-
-
-
-
-
- # [[ServiceManager的获取流程]]