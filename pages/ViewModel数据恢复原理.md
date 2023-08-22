# 使用
	- ```java
	       model=ViewModelProviders.of(this).get(NameViewModel.class);
	  
	       ViewModelProviders.of 为废弃了
	       实际使用new ViewModelProvider(activity);，of里最终使用的也是new ViewModelProvider(activity)
	  ```
	-
- # 源码流程
	- ## 常用函数
	  collapsed:: true
		- ```java
		  
		      /**
		       * 系统配置发生变更时调用
		       * @return
		       */
		      @Nullable
		      @Override
		      public Object onRetainCustomNonConfigurationInstance() {
		          return super.onRetainCustomNonConfigurationInstance();
		      }
		  
		      @Nullable
		      @Override
		      public Object getLastNonConfigurationInstance() {
		          return super.getLastNonConfigurationInstance();
		      }
		  ```
		- ## 追溯onRetainCustomNonConfigurationInstance
			- 在ComponentActivity 的onRetainNonConfigurationInstance 里调用的
				- 代码
					- ```java
					  
					  public final Object onRetainNonConfigurationInstance() {
					    // Maintain backward compatibility.
					    Object custom = onRetainCustomNonConfigurationInstance();
					  
					    ViewModelStore viewModelStore = mViewModelStore;
					    if (viewModelStore == null) {
					      // No one called getViewModelStore(), so see if there was an existing
					      // ViewModelStore from our last NonConfigurationInstance
					      NonConfigurationInstances nc =
					        (NonConfigurationInstances) getLastNonConfigurationInstance();
					      if (nc != null) {
					        viewModelStore = nc.viewModelStore;
					      }
					    }
					  
					    if (viewModelStore == null && custom == null) {
					      return null;
					    }
					  
					    NonConfigurationInstances nci = new NonConfigurationInstances();
					    nci.custom = custom;
					    nci.viewModelStore = viewModelStore;
					    return nci;
					  }
					    
					  ```
				- onRetainNonConfigurationInstance  是从 Activity的retainNonConfigurationInstances方法调用的
				  collapsed:: true
					- ```java
					  NonConfigurationInstances retainNonConfigurationInstances() {
					          Object activity = onRetainNonConfigurationInstance();
					          HashMap<String, Object> children = onRetainNonConfigurationChildInstances();
					          FragmentManagerNonConfig fragments = mFragments.retainNestedNonConfig();
					  
					          // We're already stopped but we've been asked to retain.
					          // Our fragments are taken care of but we need to mark the loaders for retention.
					          // In order to do this correctly we need to restart the loaders first before
					          // handing them off to the next activity.
					          mFragments.doLoaderStart();
					          mFragments.doLoaderStop(true);
					          ArrayMap<String, LoaderManager> loaders = mFragments.retainLoaderNonConfig();
					  
					          if (activity == null && children == null && fragments == null && loaders == null
					                  && mVoiceInteractor == null) {
					              return null;
					          }
					  
					          NonConfigurationInstances nci = new NonConfigurationInstances();
					          nci.activity = activity;
					          nci.children = children;
					          nci.fragments = fragments;
					          nci.loaders = loaders;
					          if (mVoiceInteractor != null) {
					              mVoiceInteractor.retainInstance();
					              nci.voiceInteractor = mVoiceInteractor;
					          }
					          return nci;
					      }
					  ```
				- 再后边就是通过binder机制
			- 内部干的事情
				-
	- ## 入口函数of
	  collapsed:: true
		- ```java
		  @Deprecated
		  @NonNull
		  @MainThread
		  public static ViewModelProvider of(@NonNull FragmentActivity activity) {
		    return new ViewModelProvider(activity);
		  }
		  ```
		- ## 构造
			- ```java
			  public ViewModelProvider(@NonNull ViewModelStoreOwner owner) {
			    this(owner.getViewModelStore(), owner instanceof HasDefaultViewModelProviderFactory
			         ? ((HasDefaultViewModelProviderFactory) owner).getDefaultViewModelProviderFactory()
			         : NewInstanceFactory.getInstance());
			  }
			  ```
		- owner.getViewModelStore(),,,ViewModelStoreOwner接口看其实现类
		  collapsed:: true
			- ```java
			  public interface ViewModelStoreOwner {
			      /**
			       * Returns owned {@link ViewModelStore}
			       *
			       * @return a {@code ViewModelStore}
			       */
			      @NonNull
			      ViewModelStore getViewModelStore();
			  }
			  ```
		- ComponentActivity的getViewModelStore方法[[ViewModelStore]]
		  collapsed:: true
			- ```java
			     @NonNull
			      @Override
			      public ViewModelStore getViewModelStore() {
			          if (getApplication() == null) {
			              throw new IllegalStateException("Your activity is not yet attached to the "
			                      + "Application instance. You can't request ViewModel before onCreate call.");
			          }
			          ensureViewModelStore();
			          return mViewModelStore;
			      }
			  
			  
			      void ensureViewModelStore() {
			          if (mViewModelStore == null) {
			              NonConfigurationInstances nc =
			                      (NonConfigurationInstances) getLastNonConfigurationInstance();
			              // 第一次获取为null
			              if (nc != null) {
			                  // Restore the ViewModelStore from NonConfigurationInstances
			                  mViewModelStore = nc.viewModelStore;
			              }
			              if (mViewModelStore == null) {
			                  mViewModelStore = new ViewModelStore();
			              }
			          }
			      }
			  ```
			- 第一次为null.调用[[getLastNonConfigurationInstance]] 去获取,第一次去获取为null
			- nc 也为null
			- 则 new ViewModelStore()
		- > 小结，第一次调用of，就是创建存储ViewModel的集合
	- ## get函数
	  collapsed:: true
		- ViewModelProvider 的get 代码
			- ```java
			       
			     private static final String DEFAULT_KEY =
			              "androidx.lifecycle.ViewModelProvider.DefaultKey";
			  
			  // 入参为ViewModel 的class
			      @NonNull
			      @MainThread
			      public <T extends ViewModel> T get(@NonNull Class<T> modelClass) {
			          // class 全包名字
			          String canonicalName = modelClass.getCanonicalName();
			          if (canonicalName == null) {
			              throw new IllegalArgumentException("Local and anonymous classes can not be ViewModels");
			          }
			          // 拼接
			          return get(DEFAULT_KEY + ":" + canonicalName, modelClass);
			      }
			  ```
		- get。传入key = DEFAULT_KEY + ":" + canonicalName  去ViewModelStore  map里去拿
		  collapsed:: true
			- ```java
			      
			      @NonNull
			      @MainThread
			      public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
			          // 第一次肯定取不到，第二次才有
			          ViewModel viewModel = mViewModelStore.get(key);
			  
			           // 如果取到的是 我们传入的class
			          if (modelClass.isInstance(viewModel)) {
			              if (mFactory instanceof OnRequeryFactory) {
			                  ((OnRequeryFactory) mFactory).onRequery(viewModel);
			              }
			              return (T) viewModel;
			          } else {
			              //noinspection StatementWithEmptyBody
			              if (viewModel != null) {
			                  // TODO: log a warning.
			              }
			          }
			          if (mFactory instanceof KeyedFactory) {
			              viewModel = ((KeyedFactory) mFactory).create(key, modelClass);
			          } else {
			              viewModel = mFactory.create(modelClass);
			          }
			          mViewModelStore.put(key, viewModel);
			          return (T) viewModel;
			      }
			  ```
			- ## 第一次取
			  collapsed:: true
				- 第一次取走get，取不到，通过工厂创建create。。 为抽象类工厂KeyedFactory
					- ```java
					  @NonNull
					  public abstract <T extends ViewModel> T create(@NonNull String key,
					                                                 @NonNull Class<T> modelClass);
					  ```
				- 看其子类SavedStateViewModelFactory的 Create方法
				  collapsed:: true
					- 代码
					  collapsed:: true
						- ```java
						      @NonNull
						      @Override
						      public <T extends ViewModel> T create(@NonNull String key, @NonNull Class<T> modelClass) {
						          boolean isAndroidViewModel = AndroidViewModel.class.isAssignableFrom(modelClass);
						          // 先获取viewModel  的 构造方法。反射
						          Constructor<T> constructor;
						          if (isAndroidViewModel && mApplication != null) {
						              constructor = findMatchingConstructor(modelClass, ANDROID_VIEWMODEL_SIGNATURE);
						          } else {
						              constructor = findMatchingConstructor(modelClass, VIEWMODEL_SIGNATURE);
						          }
						          // 构造方法 为null 的话。反射创建viewModel 返回
						          if (constructor == null) {
						              return mFactory.create(modelClass);
						          }
						  
						          SavedStateHandleController controller = SavedStateHandleController.create(
						                  mSavedStateRegistry, mLifecycle, key, mDefaultArgs);
						          try {
						              T viewmodel;
						              if (isAndroidViewModel && mApplication != null) {
						                  viewmodel = constructor.newInstance(mApplication, controller.getHandle());
						              } else {
						                  viewmodel = constructor.newInstance(controller.getHandle());
						              }
						              viewmodel.setTagIfAbsent(TAG_SAVED_STATE_HANDLE_CONTROLLER, controller);
						              return viewmodel;
						          } catch (IllegalAccessException e) {
						              throw new RuntimeException("Failed to access " + modelClass, e);
						          } catch (InstantiationException e) {
						              throw new RuntimeException("A " + modelClass + " cannot be instantiated.", e);
						          } catch (InvocationTargetException e) {
						              throw new RuntimeException("An exception happened in constructor of "
						                      + modelClass, e.getCause());
						          }
						      }
						  ```
					- 首次执行 反射创建ViewModel 实例
				- 然后添加到mViewModelStore 集合中
			- ## 小结：第一次执行get，会反射创建ViewModel 实例 加入ViewModelStore  的集合中
			- ## 第二次取
				- ```java
				   ViewModel viewModel = mViewModelStore.get(key);
				  ```
				- 能取到值，直接返回
				-
	- ## 屏幕发生变更时，[[#red]]==**回调ComponentActivity 的onRetainNonConfigurationInstance 方法，进行保存状态**==
		- ```java
		  
		  public final Object onRetainNonConfigurationInstance() {
		    // Maintain backward compatibility.
		    // 
		    Object custom = onRetainCustomNonConfigurationInstance();
		    // 拿到存储map,获取存储map，ViewModelStore
		    ViewModelStore viewModelStore = mViewModelStore;
		    if (viewModelStore == null) {
		      // No one called getViewModelStore(), so see if there was an existing
		      // ViewModelStore from our last NonConfigurationInstance
		      NonConfigurationInstances nc =
		        (NonConfigurationInstances) getLastNonConfigurationInstance();
		      if (nc != null) {
		        viewModelStore = nc.viewModelStore;
		      }
		    }
		  
		    if (viewModelStore == null && custom == null) {
		      return null;
		    }
		    // 创建这个 将viewModelStore 存入
		    NonConfigurationInstances nci = new NonConfigurationInstances();
		    nci.custom = custom;
		    nci.viewModelStore = viewModelStore;
		    return nci;
		  }
		    
		  ```
		- ## 配置发生变化的时候
			- 1、 获取存储map，ViewModelStore
			- 2、创建 NonConfigurationInstances 将viewModelStore 存入。并将NonConfigurationInstances返回
			  id:: 64d5d2bf-8571-4756-ba7f-0bb51711feba
		- ## 看上述返回值哪里用。看其父类 Activity 中调用上述方法的地方onRetainNonConfigurationInstance
			- 代码
				- ```java
				  NonConfigurationInstances retainNonConfigurationInstances() {
				          // 拿到 NonConfigurationInstances  实例
				          Object activity = onRetainNonConfigurationInstance();
				          HashMap<String, Object> children = onRetainNonConfigurationChildInstances();
				          FragmentManagerNonConfig fragments = mFragments.retainNestedNonConfig();
				  
				          // We're already stopped but we've been asked to retain.
				          // Our fragments are taken care of but we need to mark the loaders for retention.
				          // In order to do this correctly we need to restart the loaders first before
				          // handing them off to the next activity.
				          mFragments.doLoaderStart();
				          mFragments.doLoaderStop(true);
				          ArrayMap<String, LoaderManager> loaders = mFragments.retainLoaderNonConfig();
				  
				          if (activity == null && children == null && fragments == null && loaders == null
				                  && mVoiceInteractor == null) {
				              return null;
				          }
				  
				          NonConfigurationInstances nci = new NonConfigurationInstances();
				          nci.activity = activity;
				          nci.children = children;
				          nci.fragments = fragments;
				          nci.loaders = loaders;
				          if (mVoiceInteractor != null) {
				              mVoiceInteractor.retainInstance();
				              nci.voiceInteractor = mVoiceInteractor;
				          }
				          return nci;
				      }
				  ```
			- 这里会再次创建一个NonConfigurationInstances，将onRetainNonConfigurationInstance返回值NonConfigurationInstances，存入Activity变量中。Activity里已经保存了viewModel。
			- 通过AMS 赋值到mLastNonConfigurationInstances变量中
		- 屏幕发生旋转完再从of里取的时候
			- ```java
			      @Deprecated
			      @NonNull
			      @MainThread
			      public static ViewModelProvider of(@NonNull FragmentActivity activity) {
			          return new ViewModelProvider(activity);
			      }
			  ```
			- ```java
			      public ViewModelProvider(@NonNull ViewModelStoreOwner owner) {
			          this(owner.getViewModelStore(), owner instanceof HasDefaultViewModelProviderFactory
			                  ? ((HasDefaultViewModelProviderFactory) owner).getDefaultViewModelProviderFactory()
			                  : NewInstanceFactory.getInstance());
			      }
			  
			  ```
			- 走构造getViewModelStore,看子类ComponentActivity实现
				- ```java
				      @NonNull
				      @Override
				      public ViewModelStore getViewModelStore() {
				          if (getApplication() == null) {
				              throw new IllegalStateException("Your activity is not yet attached to the "
				                      + "Application instance. You can't request ViewModel before onCreate call.");
				          }
				          ensureViewModelStore();
				          return mViewModelStore;
				      }
				  
				      @SuppressWarnings("WeakerAccess") /* synthetic access */
				      void ensureViewModelStore() {
				          if (mViewModelStore == null) {
				              NonConfigurationInstances nc =
				                      (NonConfigurationInstances) getLastNonConfigurationInstance();
				              if (nc != null) {
				                  // Restore the ViewModelStore from NonConfigurationInstances
				                  mViewModelStore = nc.viewModelStore;
				              }
				              if (mViewModelStore == null) {
				                  mViewModelStore = new ViewModelStore();
				              }
				          }
				      }
				  ```
				- 为null 进入判断，getLastNonConfigurationInstance
					- ```java
					  @Nullable
					  public Object getLastNonConfigurationInstance() {
					    return mLastNonConfigurationInstances != null
					      ? mLastNonConfigurationInstances.activity : null;
					  }
					  ```
				- 获取屏幕翻转前mLastNonConfigurationInstances实例。activity就是上边屏幕旋转时，onRetainNonConfigurationInstance 返回值。也是个NonConfigurationInstances对象。保存 了旋转之前的 ViewModelStore 这个map
				- 获取到不为null的话。取出mViewModelStore 这个map
-
-
- ## [[ViewModel数据恢复原理总结]]