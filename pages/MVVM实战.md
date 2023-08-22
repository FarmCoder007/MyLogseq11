# [[在RecyclerView中的应用]]
- # 使用
	- Activity，Fragment持有ViewModel
- # Model
  collapsed:: true
	- # 简单单个的（可以不看）
	  collapsed:: true
		- ## 1、定义处理网络请求的逻辑-不需要分页的
		  collapsed:: true
			- 1、数据回调监听
			  collapsed:: true
				- ```java
				  public interface IBaseModelListener<DATA> {
				      void onLoadSuccess(DATA data, PagingResult... result);
				      void onLoadFail(String message);
				  }
				  ```
			- 2、网络请求的viewModel
			  collapsed:: true
				- ```java
				  public class NewsChannelModel {
				      IBaseModelListener<List<NewsChannelsBean.ChannelList>> listener;
				  
				      public NewsChannelModel(IBaseModelListener listener){
				          this.listener = listener;
				      }
				  
				      public void load(){
				          TecentNetworkApi.getService(NewsApiInterface.class)
				                  .getNewsChannels()
				                  .compose(TecentNetworkApi.getInstance().applySchedulers(new BaseObserver<NewsChannelsBean>() {
				                      @Override
				                      public void onSuccess(NewsChannelsBean newsChannelsBean) {
				                          listener.onLoadSuccess(newsChannelsBean.showapiResBody.channelList);
				                      }
				  
				                      @Override
				                      public void onFailure(Throwable e) {
				                          listener.onLoadFail(e.getMessage());
				                      }
				                  }));
				      }
				  }
				  ```
			- 3、使用,进行网络请求
			  collapsed:: true
				- ```java
				  public class HeadlineNewsFragment extends Fragment implements IBaseModelListener<List<NewsChannelsBean.ChannelList>> {
				      public HeadlineNewsFragmentAdapter adapter;
				      FragmentHomeBinding viewDataBinding;
				      private NewsChannelModel mNewsChannelModel;
				      @Override
				      public View onCreateView(@NonNull LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
				          viewDataBinding = DataBindingUtil.inflate(inflater, R.layout.fragment_home, container, false);
				          adapter = new HeadlineNewsFragmentAdapter(getChildFragmentManager());
				          viewDataBinding.tablayout.setTabMode(TabLayout.MODE_SCROLLABLE);
				          viewDataBinding.viewpager.setAdapter(adapter);
				          viewDataBinding.tablayout.setupWithViewPager(viewDataBinding.viewpager);
				          viewDataBinding.viewpager.setOffscreenPageLimit(1);
				          mNewsChannelModel = new NewsChannelModel(this);
				          mNewsChannelModel.load();
				          return viewDataBinding.getRoot();
				      }
				  
				      @Override
				      public void onLoadSuccess(List<NewsChannelsBean.ChannelList> channelLists, PagingResult... results) {
				          if(adapter != null) {
				              adapter.setChannels(channelLists);
				          }
				      }
				  
				      @Override
				      public void onLoadFail(String message) {
				          Toast.makeText(getContext(), message, Toast.LENGTH_LONG).show();
				      }
				  }
				  
				  ```
		- ## 2、需要分页的
		  collapsed:: true
			- 1、定义可以分页的viewModel
			  collapsed:: true
				- ```java
				  public class NewsListModel {
				      private IBaseModelListener<List<BaseCustomViewModel>> mListener;
				      private String mChannelId;
				      private String mChannelName;
				      private int mPage = 1;
				  
				      public NewsListModel(IBaseModelListener listener, String channelId, String channelName){
				          mListener = listener;
				          mChannelId = channelId;
				          mChannelName = channelName;
				      }
				  
				      public void refresh(){
				          mPage = 1;
				          loadNextPage();
				      }
				  
				      public void loadNextPage() {
				          TecentNetworkApi.getService(NewsApiInterface.class)
				                  .getNewsList(mChannelId,
				                          mChannelName, String.valueOf(mPage))
				                  .compose(TecentNetworkApi.getInstance().applySchedulers(new BaseObserver<NewsListBean>() {
				                      @Override
				                      public void onSuccess(NewsListBean newsChannelsBean) {
				                          List<BaseCustomViewModel> viewModels = new ArrayList<>();
				                          for(NewsListBean.Contentlist contentlist:newsChannelsBean.showapiResBody.pagebean.contentlist){
				                              if(contentlist.imageurls != null && contentlist.imageurls.size() > 0){
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
				                          mListener.onLoadSuccess(viewModels, new PagingResult(mPage == 1, viewModels.isEmpty(), viewModels.size() >= 10));
				                          mPage ++;
				                      }
				  
				                      @Override
				                      public void onFailure(Throwable e) {
				                          e.printStackTrace();
				                      }
				                  }));
				      }
				  }
				  
				  ```
			- 2、分页数据的状态，是否是第一页，是否空数据等
			  collapsed:: true
				- ```java
				  public class PagingResult {
				  
				      public PagingResult(boolean isFirstPage, boolean isEmpty, boolean hasNextPage){
				          this.isFirstPage = isFirstPage;
				          this.isEmpty = isEmpty;
				          this.hasNextPage = hasNextPage;
				      }
				  
				      public boolean isFirstPage;
				      public boolean isEmpty;
				      public boolean hasNextPage;
				  }
				  
				  ```
			- 2、使用
			  collapsed:: true
				- ```java
				  public class NewsListFragment extends Fragment implements IBaseModelListener<List<BaseCustomViewModel>> {
				      private NewsListRecyclerViewAdapter mAdapter;
				      private FragmentNewsBinding viewDataBinding;
				      private NewsListModel mNewsListModel;
				  
				      protected final static String BUNDLE_KEY_PARAM_CHANNEL_ID = "bundle_key_param_channel_id";
				      protected final static String BUNDLE_KEY_PARAM_CHANNEL_NAME = "bundle_key_param_channel_name";
				  
				      public static NewsListFragment newInstance(String channelId, String channelName) {
				          NewsListFragment fragment = new NewsListFragment();
				          Bundle bundle = new Bundle();
				          bundle.putString(BUNDLE_KEY_PARAM_CHANNEL_ID, channelId);
				          bundle.putString(BUNDLE_KEY_PARAM_CHANNEL_NAME, channelName);
				          fragment.setArguments(bundle);
				          return fragment;
				      }
				  
				      @Override
				      public View onCreateView(@NonNull LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
				          viewDataBinding = DataBindingUtil.inflate(inflater, R.layout.fragment_news, container, false);
				          mAdapter = new NewsListRecyclerViewAdapter();
				          viewDataBinding.listview.setHasFixedSize(true);
				          viewDataBinding.listview.setLayoutManager(new LinearLayoutManager(getContext()));
				          viewDataBinding.listview.setAdapter(mAdapter);
				          mNewsListModel = new NewsListModel(this,getArguments().getString(BUNDLE_KEY_PARAM_CHANNEL_ID),getArguments().getString(BUNDLE_KEY_PARAM_CHANNEL_NAME));
				          mNewsListModel.refresh();
				          viewDataBinding.refreshLayout.setOnRefreshListener(new OnRefreshListener() {
				              @Override
				              public void onRefresh(@NonNull RefreshLayout refreshLayout) {
				                  mNewsListModel.refresh();
				              }
				          });
				          viewDataBinding.refreshLayout.setOnLoadMoreListener(new OnLoadMoreListener() {
				              @Override
				              public void onLoadMore(@NonNull RefreshLayout refreshLayout) {
				                  mNewsListModel.loadNextPage();
				              }
				          });
				          return viewDataBinding.getRoot();
				      }
				      private List<BaseCustomViewModel> viewModels = new ArrayList<>();
				  
				      @Override
				      public void onLoadSuccess(List<BaseCustomViewModel> baseCustomViewModels, PagingResult... results) {
				          if(results != null && results.length > 0 && results[0].isFirstPage) {
				              viewModels.clear();
				          }
				          viewModels.addAll(baseCustomViewModels);
				          mAdapter.setData(viewModels);
				          viewDataBinding.refreshLayout.finishRefresh();
				          viewDataBinding.refreshLayout.finishLoadMore();
				      }
				  
				      @Override
				      public void onLoadFail(String message) {
				          Toast.makeText(getContext(), message, Toast.LENGTH_LONG).show();
				      }
				  }
				  
				  ```
	- # 汇总[[BaseMvvmModel]]
- # View
  collapsed:: true
	- BaseMvvmFragment
		- ```java
		  public abstract class BaseMvvmFragment<VIEW extends ViewDataBinding, VIEWMODEL extends BaseMvvmViewModel, DATA> extends Fragment implements Observer {
		  
		       protected VIEWMODEL viewModel;
		       protected VIEW viewDataBinding;
		  
		      /**
		       * 打印生命周期用
		       * @return
		       */
		       protected abstract String getFragmentTag();
		  
		      /**
		       * 布局id
		       * @return
		       */
		       public abstract @LayoutRes int getLayoutId();
		  
		      /**
		       * 创建viewModel
		       * @return
		       */
		       public abstract VIEWMODEL getViewModel();
		      private LoadService mLoadService;
		  
		      @Override
		      public void onCreate(@Nullable Bundle savedInstanceState) {
		          super.onCreate(savedInstanceState);
		          Log.d(getFragmentTag(), "onCreate.");
		      }
		  
		      @Nullable
		      @Override
		      public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
		          super.onCreateView(inflater, container, savedInstanceState);
		          Log.d(getFragmentTag(), "onCreateView.");
		          viewDataBinding = DataBindingUtil.inflate(inflater, getLayoutId(), container, false);
		          return viewDataBinding.getRoot();
		      }
		  
		      public abstract void onNetworkResponded(List<DATA> dataList, boolean isDataUpdated);
		  
		      @Override
		      public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
		          super.onViewCreated(view, savedInstanceState);
		          Log.d(getFragmentTag(), "onViewCreated.");
		          viewModel = getViewModel();
		          getLifecycle().addObserver(viewModel);
		          viewModel.dataList.observe(this, this);
		          viewModel.viewStatusLiveData.observe(this, this);
		          onViewCreated();
		      }
		  
		      protected abstract void onViewCreated();
		  
		      @Override
		      public void onActivityCreated(@Nullable Bundle savedInstanceState) {
		          super.onActivityCreated(savedInstanceState);
		          Log.d(getFragmentTag(), "Activity:" + getActivity() + " Fragment:"+this + ": " + "onActivityCreated");
		      }
		  
		      @Override
		      public void onAttach(Context context) {
		          super.onAttach(getContext());
		          Log.d(getFragmentTag(), "Activity:" + getActivity() + " Fragment:"+this + ": " + "onAttach");
		      }
		  
		      @Override
		      public void onDetach() {
		          super.onDetach();
		          Log.d(getFragmentTag(), "Activity:" + getActivity() + " Fragment:"+this + ": " + "onDetach");
		      }
		  
		      @Override
		      public void onStop() {
		          super.onStop();
		          Log.d(getFragmentTag(), "Activity:" + getActivity() + " Fragment:"+this + ": " + "onStop");
		      }
		  
		      @Override
		      public void onPause() {
		          super.onPause();
		          Log.d(getFragmentTag(), "Activity:" + getActivity() + " Fragment:"+this + ": " + "onPause");
		      }
		  
		      @Override
		      public void onResume() {
		          super.onResume();
		          Log.d(getFragmentTag(), "Activity:" + getActivity() + " Fragment:"+this + ": " + "onResume");
		      }
		  
		      @Override
		      public void onDestroy() {
		          Log.d(getFragmentTag(), "Activity:" + getActivity() + " Fragment:"+this + ": " + "onDestroy");
		          super.onDestroy();
		      }
		  
		      @Override
		      public void onDestroyView() {
		          Log.d(getFragmentTag(), "Activity:" + getActivity() + " Fragment:"+this + ": " + "onDestroyView");
		          super.onDestroyView();
		      }
		  
		      public void setLoadSir(View view) {
		          // You can change the callback on sub thread directly.
		          mLoadService = LoadSir.getDefault().register(view, new Callback.OnReloadListener() {
		              @Override
		              public void onReload(View v) {
		                  viewModel.refresh();
		              }
		          });
		      }
		  
		      /**
		       * 根据 view状态显示不同 容错ui
		       * @param o  The new data
		       */
		      @Override
		      public void onChanged(Object o) {
		          if (o instanceof ViewStatus && mLoadService != null) {
		  /*            switch ((ViewStatus) o) {
		                  case LOADING:
		                      mLoadService.showCallback(LoadingCallback.class);
		                      break;
		                  case EMPTY:
		                      mLoadService.showCallback(EmptyCallback.class);
		                      break;
		                  case SHOW_CONTENT:
		                      mLoadService.showSuccess();
		                      break;
		                  case NO_MORE_DATA:
		                      ToastUtil.show(getString(R.string.no_more_data));
		                      break;
		                  case REFRESH_ERROR:
		                      if (((ObservableArrayList) viewModel.dataList.getValue()).size() == 0) {
		                          mLoadService.showCallback(ErrorCallback.class);
		                      } else {
		                          ToastUtil.show(viewModel.errorMessage.getValue().toString());
		                      }
		                      break;
		                  case LOAD_MORE_FAILED:
		                      ToastUtil.show(viewModel.errorMessage.getValue().toString());
		                      break;
		              }*/
		             // onNetworkResponded(null, false);
		          } else if (o instanceof List) {
		              onNetworkResponded((List<DATA>)o, true);
		            //  mLoadService.showSuccess();
		          }
		      }
		  }
		  
		  ```
- # ViewModel
	- BaseMvvmViewModel
	  collapsed:: true
		- ```java
		  public abstract class BaseMvvmViewModel<MODEL extends BaseMvvmModel, DATA> extends ViewModel implements LifecycleObserver, IBaseModelListener<List<DATA>> {
		      /**
		       * 多个api 可以多个liveData
		       */
		      public MutableLiveData<List<DATA>> dataList = new MutableLiveData<>();
		  
		      /**
		       * 实例化 model
		       */
		      protected MODEL model;
		      /**
		       * 请求状态 liveData
		       */
		      public MutableLiveData<ViewStatus> viewStatusLiveData = new MutableLiveData();
		      /**
		       * 错误livedata
		       */
		      public MutableLiveData<String> errorMessage = new MutableLiveData<>();
		  
		      public BaseMvvmViewModel() {
		  
		      }
		  
		      /**
		       * 创建 model
		       * @return
		       */
		      public abstract MODEL createModel();
		  
		      /**
		       * 使用model  触发刷新
		       * 改变view 当前状态
		       */
		      public void refresh() {
		          viewStatusLiveData.setValue(ViewStatus.LOADING);
		          createAndRegisterModel();
		          if (model != null) {
		              model.refresh();
		          }
		      }
		  
		      /**
		       * 创建并且注册 model
		       */
		      private void createAndRegisterModel() {
		          if (model == null) {
		              model = createModel();
		              if (model != null) {
		                  model.register(this);
		              } else {
		                  // Throw exception.
		              }
		          }
		      }
		  
		      /**
		       * 获取缓存数据
		       */
		      public void getCachedDataAndLoad() {
		          viewStatusLiveData.setValue(ViewStatus.LOADING);
		          createAndRegisterModel();
		          if (model != null) {
		              model.getCachedDataAndLoad();
		          }
		      }
		  
		      /**
		       * 加载下一页
		       */
		      public void loadNextPage() {
		          createAndRegisterModel();
		          if (model != null) {
		              model.loadNextPage();
		          }
		      }
		  
		      /**
		       * Jetpack  viewmodel 的方法，页面销毁时，回调，取消网络请求
		       */
		      @Override
		      protected void onCleared() {
		          super.onCleared();
		          if (model != null) {
		              model.cancel();
		          }
		      }
		  
		      /**
		       * 数据加载成功 分发
		       * @param model
		       * @param data
		       * @param pagingResult
		       */
		      @Override
		      public void onLoadSuccess(BaseMvvmModel model, List<DATA> data, PagingResult... pagingResult) {
		          // 是分页 加载成功
		          if (model.isPaging()) {
		              // 数据为空
		              if (pagingResult[0].isEmpty) {
		                  if (pagingResult[0].isFirstPage) {// 第一页
		                      viewStatusLiveData.postValue(ViewStatus.EMPTY);// 数据为空
		                  } else {
		                      viewStatusLiveData.postValue(ViewStatus.NO_MORE_DATA); // 没有更多
		                  }
		              } else { // 数据不为空
		                  if (pagingResult[0].isFirstPage) {
		                      dataList.postValue(data); // 发送数据
		                  } else {
		                      dataList.getValue().addAll(data); // 添加到全部数据里
		                      dataList.postValue(dataList.getValue());
		                  }
		                  viewStatusLiveData.postValue(ViewStatus.SHOW_CONTENT);
		              }
		          } else {
		              dataList.postValue(data);
		              viewStatusLiveData.postValue(ViewStatus.SHOW_CONTENT);
		          }
		      }
		  
		      /**
		       * 加载失败
		       * @param model
		       * @param message
		       * @param result
		       */
		      @Override
		      public void onLoadFail(BaseMvvmModel model, String message, PagingResult... result) {
		          errorMessage.postValue(message);
		          if (result[0].isFirstPage) { // 是首页 刷新错误
		              viewStatusLiveData.postValue(ViewStatus.REFRESH_ERROR);
		          } else {
		              viewStatusLiveData.postValue(ViewStatus.LOAD_MORE_FAILED); // 加载更多错误
		          }
		      }
		  
		      @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
		      private void onResume() {
		          // onresume 时 数据不存在就拉取数据
		          if(dataList == null || dataList.getValue() == null || dataList.getValue().size() == 0) {
		              createAndRegisterModel();
		              model.getCachedDataAndLoad();
		          } else { // 否则直接从已存在的数据拿
		              dataList.postValue(dataList.getValue());
		              viewStatusLiveData.postValue(viewStatusLiveData.getValue());
		          }
		      }
		  }
		  
		  ```
	- 1、创建model
	- 2、监听model里的数据回调
	- 3、
-