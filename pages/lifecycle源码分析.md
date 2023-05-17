- 序言
	- 之前对于lifecycle三件套的理解只存在与CV使用和看了几篇文章的层面上。这次在开发达人版app的时候尝试使用了下，发下的确好用，所以在此从源码角度上总结学习一下。
- 为什么用Lifecycle组件
	- Lifecycle生命周期可感知，可以大大降低开发成本和内存泄漏的风险。试想MVP那套框架，你需要把Activity或fragment的生命周期一步步的注入到你的框架里是多么痛苦。
	- LiveData替换Rxbus的利器。LiveData也可以进行事件传递，在结合了Lifecycle的生命周期感知之后，你的开发难度将大大降低。你将不用考虑事件接收方已经被销毁这种边界问题。
	- viewModel数据存储共享好帮手。如果没有viewModel要实现数据共享无非以下几种方式，写全局静态类静态方法、搞SP存储。这些方法各有优缺，但是都有一个蛋疼的问题就是内存的管理。而使用viewModel则会帮你解决这些烦恼。
- # 用法与源码分析
- ## ViewModel解析
	- 对于activty或fragment很简单，直接用ViewModelProviders就可以构造出来。看参数大概可以知道传入activity那么就跟activity生命周期绑定，传入fragment就跟fragment绑定。
	- ```
	   MyViewModel model = ViewModelProviders.of(activity).get(MyViewModel.class);
	  或
	   MyViewModel model = ViewModelProviders.of(fragment).get(MyViewModel.class);
	  ```
	- 我们点进去看下源码这几个方法做了什么。
	- ```
	  @MainThread
	  public static ViewModelProvider of(@NonNull FragmentActivity activity,@Nullable Factory factory) {
	    Application application = checkApplication(activity);
	    if (factory == null) {
	        factory = ViewModelProvider.AndroidViewModelFactory.getInstance(application);
	    }
	    return new ViewModelProvider(activity.getViewModelStore(), factory);
	  }
	  ```
	  可以看到构造ViewModelProvider的时候传递了activity/fragment的ViewModelStore和factory两个东西。
	- 那ViewModelStore是如何实现的，activity又是如何管理的呢？查看ViewModelStore的源码，发现内部只是创建了一个map集合构造了，用于存储和管理。所以逻辑很简单。
	- ```
	  public class ViewModelStore {
	  
	      private final HashMap<String, ViewModel> mMap = new HashMap<>();
	  
	      final void put(String key, ViewModel viewModel) {
	          ViewModel oldViewModel = mMap.put(key, viewModel);
	          if (oldViewModel != null) {
	              oldViewModel.onCleared();
	          }
	      }
	  
	      final ViewModel get(String key) {
	          return mMap.get(key);
	      }
	  
	      /**
	       *  Clears internal storage and notifies ViewModels that they are no longer used.
	       */
	      public final void clear() {
	          for (ViewModel vm : mMap.values()) {
	              vm.onCleared();
	          }
	          mMap.clear();
	      }
	  }
	  ```
	  在接着在activty里查看ViewModelStore的调用，一共有三处。我们先从最简单的来。
	  
	  第一处：
	  
	  ```
	      @Override
	      protected void onDestroy() {
	          super.onDestroy();
	  
	          if (mViewModelStore != null && !isChangingConfigurations()) {
	              mViewModelStore.clear();
	          }
	  
	          mFragments.dispatchDestroy();
	      }
	  ```
	  是不是很简单，对应上边ViewModelStore源码，我们知道activity销毁的时候会先调用持有的ViewModel的clear，然后在将持有的viewModel释放掉。也就是说我们只用关心在自己ViewModel的clear方法调用的时候释放掉自己其他的内存引用，但不用关心在activty销毁的时候释自己的ViewModel。
	  
	  接下来两处是个相关联的我们放一起看：
	  
	  ```
	      @Override
	      public final Object onRetainNonConfigurationInstance() {
	          Object custom = onRetainCustomNonConfigurationInstance();
	  
	          FragmentManagerNonConfig fragments = mFragments.retainNestedNonConfig();
	  
	          if (fragments == null && mViewModelStore == null && custom == null) {
	              return null;
	          }
	  
	          NonConfigurationInstances nci = new NonConfigurationInstances();
	          nci.custom = custom;
	          nci.viewModelStore = mViewModelStore;
	          nci.fragments = fragments;
	          return nci;
	      }
	  
	      @Override
	      protected void onCreate(@Nullable Bundle savedInstanceState) {
	          mFragments.attachHost(null /*parent*/);
	  
	          super.onCreate(savedInstanceState);
	  
	          NonConfigurationInstances nc =
	                  (NonConfigurationInstances) getLastNonConfigurationInstance();
	          if (nc != null && nc.viewModelStore != null && mViewModelStore == null) {
	              mViewModelStore = nc.viewModelStore;
	          }
	      }
	  ```
	  看代码我们可以知道这是一个状态存储和恢复的过程。在activity重建的时候会拿出上次的viewModelStore进行赋值。
	  
	  这就有另一个知识点了onRetainNonConfigurationInstance是什么鬼，和另一个状态存储方法onSaveInstanceState是什么关系？
	  
	  NonConfigurationInstance可以保存实例对象，甚至之前的activity，onSaveInstanceState只能保存数据Bundle
	  
	  我们接着看构造ViewModelProvider的另一个东西factory。
	  ```
	  /**
	   * Simple factory, which calls empty constructor on the give class.
	   */
	  public static class NewInstanceFactory implements Factory {
	  
	      @SuppressWarnings("ClassNewInstance")
	      @NonNull
	      @Override
	      public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
	          //noinspection TryWithIdenticalCatches
	          try {
	              return modelClass.newInstance();
	          } catch (InstantiationException e) {
	              throw new RuntimeException("Cannot create an instance of " + modelClass, e);
	          } catch (IllegalAccessException e) {
	              throw new RuntimeException("Cannot create an instance of " + modelClass, e);
	          }
	      }
	  }
	  
	  public static class AndroidViewModelFactory extends ViewModelProvider.NewInstanceFactory {
	  
	      private static AndroidViewModelFactory sInstance;
	  
	      /**
	       * Retrieve a singleton instance of AndroidViewModelFactory.
	       *
	       * @param application an application to pass in {@link AndroidViewModel}
	       * @return A valid {@link AndroidViewModelFactory}
	       */
	      @NonNull
	      public static AndroidViewModelFactory getInstance(@NonNull Application application) {
	          if (sInstance == null) {
	              sInstance = new AndroidViewModelFactory(application);
	          }
	          return sInstance;
	      }
	  
	      private Application mApplication;
	  
	      /**
	       * Creates a {@code AndroidViewModelFactory}
	       *
	       * @param application an application to pass in {@link AndroidViewModel}
	       */
	      public AndroidViewModelFactory(@NonNull Application application) {
	          mApplication = application;
	      }
	  
	      @NonNull
	      @Override
	      public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
	          if (AndroidViewModel.class.isAssignableFrom(modelClass)) {
	              //noinspection TryWithIdenticalCatches
	              try {
	                  return modelClass.getConstructor(Application.class).newInstance(mApplication);
	              } catch (NoSuchMethodException e) {
	                  throw new RuntimeException("Cannot create an instance of " + modelClass, e);
	              } catch (IllegalAccessException e) {
	                  throw new RuntimeException("Cannot create an instance of " + modelClass, e);
	              } catch (InstantiationException e) {
	                  throw new RuntimeException("Cannot create an instance of " + modelClass, e);
	              } catch (InvocationTargetException e) {
	                  throw new RuntimeException("Cannot create an instance of " + modelClass, e);
	              }
	          }
	          return super.create(modelClass);
	      }
	  }
	  ```
	  看代码我们可以知道这是对传入的字节码进行实例化构造。如果是AndroidViewModel，那么会调用带application的构造方法进行实例化，否则就用默认的构造函数实例化。
	  
	  那么问题又来了
	  
	  1. AndroidViewModel和ViewModel有什么区别，什么时候用AndroidViewModel什么时候用AndroidViewModel?
	  
	  由于 ViewModel 生命周期可能长与 activity 生命周期，所以为了避免内存泄漏 Google 禁止在 ViewModel 中持有 Context 或 activity 或 view 的引用。
	  
	  
	  2. 思考下，如果我想要个跟application生命周期绑定的viewModel怎么办？
	  
	  能不能直接new一个全局的ViewModel
	  
	  下边来张图我们总结下刚才学到的这些。
	-