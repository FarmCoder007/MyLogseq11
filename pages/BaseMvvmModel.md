# 数据请求流程：（网络和缓存）
	- rxjava ->BaseObserver(MvvmDataObserver)->BaseMvvmModel->IBaseModelListener->viewmodel层
- ## 最终代码BaseMvvmModel
  collapsed:: true
	- ```java
	  public abstract class BaseMvvmModel<NETWORK_DATA, RESULT_DATA> implements MvvmDataObserver<NETWORK_DATA>{
	  
	      private CompositeDisposable compositeDisposable;
	      protected int mPage = 1;
	      /**
	       *  弱引用回调监听
	       */
	      protected WeakReference<IBaseModelListener> mReferenceIBaseModelListener;
	      /**
	       *  是否需要分页
	       */
	      private boolean mIsPaging;
	      /**
	       *  分页时初始的页号，比如数据我就想从第3页开始
	       */
	      private final int INIT_PAGE_NUMBER;
	      /**
	       *  是否是正在加载
	       */
	      private boolean mIsLoading;
	      /**
	       *  缓存数据到 sp 的key
	       */
	      private String mCachedPreferenceKey;
	      /**
	       * apk 内置数据，无网络 无缓存用
	       */
	      private String mApkPredefinedData;
	  
	      public BaseMvvmModel(boolean isPaging, String cachedPreferenceKey, String apkPredefinedData, int... initPageNumber) {
	          this.mIsPaging = isPaging;
	          if (isPaging && initPageNumber != null && initPageNumber.length > 0) {
	              INIT_PAGE_NUMBER = initPageNumber[0];
	          } else {
	              INIT_PAGE_NUMBER = -1;
	          }
	          this.mCachedPreferenceKey = cachedPreferenceKey;
	          this.mApkPredefinedData = apkPredefinedData;
	      }
	  
	      /**
	       * 注册监听
	       * @param listener
	       */
	      public void register(IBaseModelListener listener) {
	          if (listener != null) {
	              mReferenceIBaseModelListener = new WeakReference<>(listener);
	          }
	      }
	  
	  
	      /**
	       *  刷新
	       */
	      public void refresh() {
	          // Need to throw exception if register is not called;
	          if (!mIsLoading) {
	              if (mIsPaging) {
	                  mPage = INIT_PAGE_NUMBER;
	              }
	              mIsLoading = true;
	              load();
	          }
	      }
	  
	      /**
	       * 加载下一页
	       */
	      public void loadNextPage() {
	          // Need to throw exception if register is not called;
	          if (!mIsLoading) {
	              mIsLoading = true;
	              load();
	          }
	      }
	  
	      /**
	       * 是否需要更新 model
	       * @param cachedTimeSlot
	       * @return
	       */
	      protected boolean isNeedToUpdate(long cachedTimeSlot) {
	          return true;
	      }
	  
	  
	      /**
	       *  获取缓存数据，同时加载网络
	       */
	      public void getCachedDataAndLoad(){
	          if(!mIsLoading) {
	              mIsLoading = true;
	              if(mCachedPreferenceKey != null){
	                  String saveDataString = BasicDataPreferenceUtil.getInstance().getString(mCachedPreferenceKey);
	                  // 缓存数据 不为空
	                  if(!TextUtils.isEmpty(saveDataString)) {
	                      try {
	                          NETWORK_DATA savedData = new Gson().fromJson(new JSONObject(saveDataString).getString("data"), (Class<NETWORK_DATA>) GenericUtils.getGenericType(this));
	                          if(savedData != null) {
	                              // 回调成功 来自缓存
	                              onSuccess(savedData, true);
	                          }
	                          long timeSlot = Long.parseLong(new JSONObject(saveDataString).getString("updateTimeInMillis"));
	                          if(isNeedToUpdate(timeSlot)) {
	                              // 需要更新缓存，就是load()
	                              load();
	                              return;
	                          }
	                      } catch (JSONException e) {
	                          Log.e("BaseMvvmModel",e.getMessage());
	                          //e.printStackTrace();
	                      }
	                  }
	  
	                  // 没有缓存 走这里，使用内置数据
	                  if(mApkPredefinedData != null) {
	                      NETWORK_DATA savedData = new Gson().fromJson(mApkPredefinedData, (Class<NETWORK_DATA>) GenericUtils.getGenericType(this));
	                      if(savedData != null) {
	                          onSuccess(savedData, true);
	                      }
	                  }
	              }
	              // 同时加载网络请求
	              load();
	          }
	      }
	  
	  
	      public abstract void load();
	  
	      /**
	       *  通知时候需要知道是否来自缓存
	       * @param networkData 返回的网络完整数据
	       * @param resultData  可直接使用的数据
	       * @param isFromCache 是否来自缓存
	       */
	      protected void notifyResultToListener(NETWORK_DATA networkData, RESULT_DATA resultData, boolean isFromCache) {
	          IBaseModelListener listener = mReferenceIBaseModelListener.get();
	          if (listener != null) {
	              // notify
	              if (mIsPaging) {
	                  listener.onLoadSuccess(this, resultData, new PagingResult(mPage == INIT_PAGE_NUMBER, resultData == null ? true : ((List) resultData).isEmpty(), ((List) resultData).size() > 0));
	              } else {
	                  listener.onLoadSuccess(this, resultData);
	              }
	  
	              // // 保存缓存
	              if(mIsPaging) {
	                  if(mCachedPreferenceKey != null && mPage == INIT_PAGE_NUMBER && !isFromCache){
	                      saveDataToPreference(networkData);
	                  }
	              } else {
	                  if(mCachedPreferenceKey != null && !isFromCache){
	                      saveDataToPreference(networkData);
	                  }
	              }
	  
	              //   // 如果是分页，还有下一页，更新mPage
	              if (mIsPaging && !isFromCache) {
	                  if (resultData != null && ((List) resultData).size() > 0) {
	                      mPage++;
	                  }
	              }
	          }
	          if(!isFromCache) {
	              mIsLoading = false;
	          }
	      }
	  
	      /**
	       *  通知加载失败
	       * @param errorMessage
	       */
	      protected void loadFail(final String errorMessage) {
	          IBaseModelListener listener = mReferenceIBaseModelListener.get();
	          if (listener != null) {
	              if (mIsPaging) {
	                  listener.onLoadFail(this, errorMessage, new PagingResult(mPage == INIT_PAGE_NUMBER, true, false));
	              } else {
	                  listener.onLoadFail(this, errorMessage);
	              }
	          }
	          mIsLoading = false;
	      }
	  
	      /**
	       * 缓存线上数据
	       * @param data
	       */
	      protected void saveDataToPreference(NETWORK_DATA data) {
	          if(data != null) {
	              BaseCachedData<NETWORK_DATA> cachedData = new BaseCachedData<>();
	              cachedData.data = data;
	              cachedData.updateTimeInMillis = System.currentTimeMillis();
	              BasicDataPreferenceUtil.getInstance().setString(mCachedPreferenceKey, new Gson().toJson(cachedData));
	          }
	      }
	  
	      /**
	       *  取消网络请求
	       */
	      public void cancel() {
	          if (compositeDisposable != null && !compositeDisposable.isDisposed()) {
	              compositeDisposable.dispose();
	          }
	      }
	  
	  
	      /**
	       *  拿到rx 本次请求的 Disposable
	       * @param d
	       */
	      public void addDisposable(Disposable d) {
	          if (d == null) {
	              return;
	          }
	  
	          if (compositeDisposable == null) {
	              compositeDisposable = new CompositeDisposable();
	          }
	  
	          compositeDisposable.add(d);
	      }
	  
	      /**
	       * 是否是分页
	       * @return
	       */
	      public boolean isPaging(){
	          return mIsPaging;
	      }
	  }
	  
	  ```
- ## 1、数据的监听（网络和缓存）
	- MvvmDataObserver：回调给具体Model的
	  collapsed:: true
		- ```java
		  /**
		   *  数据的监听：从网络和缓存回调回来
		   * @param <F>
		   */
		  public interface MvvmDataObserver<F> {
		      /**
		       * 区分 缓存和网络 2部分数据来源
		       * @param t
		       * @param isFromCache
		       */
		      void onSuccess(F t, boolean isFromCache);
		      void onFailure(Throwable e);
		  }
		  
		  ```
	- IBaseModelListener:回调给具体的viewModel
	  collapsed:: true
		- ```java
		  /**
		   *   数据回调给ViewModel
		   * @param <DATA>
		   */
		  public interface IBaseModelListener<DATA> {
		      void onLoadSuccess(BaseMvvmModel model, DATA data, PagingResult... result);
		      void onLoadFail(BaseMvvmModel model, String message, PagingResult... result);
		  }
		  
		  ```
- ## 2、分页逻辑+刷新+加载更多
  collapsed:: true
	- PagingResult记录分页状态
		- ```java
		  /**
		   *  记录分页状态
		   */
		  public class PagingResult {
		  
		      public PagingResult(boolean isFirstPage, boolean isEmpty, boolean hasNextPage){
		          this.isFirstPage = isFirstPage;
		          this.isEmpty = isEmpty;
		          this.hasNextPage = hasNextPage;
		      }
		  
		      public boolean isFirstPage; // 是否第一页
		      public boolean isEmpty;// 是否 为空
		      public boolean hasNextPage;// 是否有下一页
		  }
		  
		  ```
- ## 3、缓存数据处理
	- 1、BaseObserver：就是添加一层回调，标识是从网络返回的
	  collapsed:: true
		- ```java
		  /**
		   * 主要作用：就是添加一层回调，标识是从网络返回的
		   * @param <T>
		   */
		  public class BaseObserver<T> implements Observer<T> {
		      /**
		       *  持有 BaseMvvmModel
		       */
		      BaseMvvmModel baseModel;
		  
		      /**
		       *  数据监听。回调给具体ViewModel的
		       */
		      MvvmDataObserver<T> mvvmDataObserver;
		  
		  
		      public BaseObserver(BaseMvvmModel baseModel, MvvmDataObserver<T> mvvmDataObserver) {
		          this.baseModel = baseModel;
		          this.mvvmDataObserver = mvvmDataObserver;
		      }
		  
		      /**
		       *  添加Disposable 到 BaseMvvmModel 取消请求用
		       * @param d the Disposable instance whose {@link Disposable#dispose()} can
		       * be called anytime to cancel the connection
		       */
		      @Override
		      public void onSubscribe(Disposable d) {
		          if(baseModel != null){
		              baseModel.addDisposable(d);
		          }
		      }
		  
		      /**
		       *  数据回调回来后，标识从网络返回
		       * @param t
		       *          the item emitted by the Observable
		       */
		      @Override
		      public void onNext(T t) {
		          mvvmDataObserver.onSuccess(t, false);
		      }
		  
		      /**
		       *  失败回调
		       * @param e
		       *          the exception encountered by the Observable
		       */
		      @Override
		      public void onError(Throwable e) {
		          if(e instanceof ExceptionHandle.ResponeThrowable){
		              mvvmDataObserver.onFailure(e);
		          } else {
		              mvvmDataObserver.onFailure(new ExceptionHandle.ResponeThrowable(e, ExceptionHandle.ERROR.UNKNOWN));
		          }
		      }
		  
		      @Override
		      public void onComplete() {
		  
		      }
		  }
		  
		  ```
	- 2、BaseCachedData
	  collapsed:: true
		- ```java
		  /**
		   * 缓存数据用
		   * @param <DATA>
		   */
		  public class BaseCachedData<DATA> {
		      @SerializedName("updateTimeInMillis")
		      @Expose
		      public long updateTimeInMillis;
		  
		      @SerializedName("data")
		      @Expose
		      public DATA data;
		  }
		  ```
- sp工具
  collapsed:: true
	- ```java
	  public class BasicDataPreferenceUtil extends BasePreferences {
	      private static final String BASIC_DATA_PREFERENCE_FILE_NAME = "network_api_module_basic_data_preference";
	      private static BasicDataPreferenceUtil sInstance;
	  
	      public static BasicDataPreferenceUtil getInstance() {
	          if (sInstance == null) {
	              synchronized (BasicDataPreferenceUtil.class) {
	                  if (sInstance == null) {
	                      sInstance = new BasicDataPreferenceUtil();
	                  }
	              }
	          }
	          return sInstance;
	      }
	  
	      @Override
	      protected String getSPFileName() {
	          return BASIC_DATA_PREFERENCE_FILE_NAME;
	      }
	  }
	  
	  ```
- # 实际使用
	- ```java
	  public class NewsListModel extends BaseMvvmModel<NewsListBean, List<BaseCustomViewModel>> {
	      private String mChannelId;
	      private String mChannelName;
	  
	      public NewsListModel(String channelId, String channelName) {
	          super(true, channelId + channelName + "_preference_key", null,1);
	          mChannelId = channelId;
	          mChannelName = channelName;
	      }
	  
	      @Override
	      public void load() {
	          TecentNetworkApi.getService(NewsApiInterface.class)
	                  .getNewsList(mChannelId,
	                          mChannelName, String.valueOf(mPage))
	                  .compose(TecentNetworkApi.getInstance().applySchedulers(new BaseObserver<NewsListBean>(this, this)));
	      }
	  
	      @Override
	      public void onSuccess(NewsListBean newsListBean, boolean isFromCache) {
	          List<BaseCustomViewModel> viewModels = new ArrayList<>();
	          for (NewsListBean.Contentlist contentlist : newsListBean.showapiResBody.pagebean.contentlist) {
	              if (contentlist.imageurls != null && contentlist.imageurls.size() > 0) {
	                  PictureTitleViewModel pictureTitleViewModel = new PictureTitleViewModel();
	                  pictureTitleViewModel.pictureUrl = contentlist.imageurls.get(0).url;
	                  pictureTitleViewModel.jumpUri = contentlist.link;
	                  pictureTitleViewModel.title = contentlist.title;
	                  viewModels.add(pictureTitleViewModel);
	              } else {
	                  TitleViewModel titleViewModel = new TitleViewModel();
	                  titleViewModel.jumpUri = contentlist.link;
	                  titleViewModel.title = contentlist.title;
	                  viewModels.add(titleViewModel);
	              }
	          }
	          notifyResultToListener(newsListBean, viewModels, isFromCache);
	      }
	  
	      @Override
	      public void onFailure(Throwable e) {
	          loadFail(e.getMessage());
	      }
	  }
	  
	  ```