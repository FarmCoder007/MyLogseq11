# 1、将ItemView中使用自定义view抽离
	- ## 1-1、title_view.xml，声明databinding绑定数据
	  collapsed:: true
		- ```xml
		  <?xml version="1.0" encoding="utf-8"?>
		  <layout xmlns:android="http://schemas.android.com/apk/res/android">
		  
		      <data>
		          <import type="android.view.View" />
		          <import type="android.text." />
		          <import type="androidx.databindinTextUtilsg.ObservableField" />
		          <variable
		              name="viewModel"
		              type="com.xiangxue.news.homefragment.newslist.views.titleview.TitleViewModel" />
		      </data>
		  
		      <LinearLayout
		          android:layout_width="match_parent"
		          android:layout_height="wrap_content"
		          android:animateLayoutChanges="true"
		          android:orientation="vertical">
		  
		          <TextView
		              android:layout_margin="16dp"
		              android:id="@+id/item_title"
		              android:layout_width="match_parent"
		              android:layout_height="wrap_content"
		              android:ellipsize="end"
		              android:visibility="@{TextUtils.isEmpty(viewModel.title)?View.GONE:View.VISIBLE}"
		              android:maxLines="2"
		              android:text="@{viewModel.title}"
		              android:gravity="center_vertical"
		              android:textColor="#303030"
		              android:textSize="16sp"
		              android:textStyle="bold"/>
		  
		          <View
		              android:layout_width="match_parent"
		              android:layout_height="1px"
		              android:background="#303030" />
		      </LinearLayout>
		  </layout>
		  ```
	- ## 1-2、自定义view的处理,
		- 1、将xml inflate 出来
		- 2、拿到mViewModel，设置给 TitleViewBinding。外部mViewModel数据变化，会自动更新到ui上
		- ```java
		  public interface BaseCustomView<D extends BaseCustomViewModel> {
		      void setData(D data);
		  }
		  
		  
		  public class TitleView extends LinearLayout implements BaseCustomView<TitleViewModel> {
		      // xml里使用了 Databinding  自动生成的TitleViewBinding
		      private TitleViewBinding mBinding;
		      private  TitleViewModel mViewModel;
		      public TitleView(Context context) {
		          super(context);
		          init();
		      }
		  
		      public void init(){
		          // 获取inflater
		          LayoutInflater inflater = (LayoutInflater) getContext().getSystemService(Context.LAYOUT_INFLATER_SERVICE);
		          // 通过DataBindingUtil 加载布局
		          mBinding = DataBindingUtil.inflate(inflater, R.layout.title_view, this, false);
		          mBinding.getRoot().setOnClickListener(new OnClickListener() {
		              @Override
		              public void onClick(View v) {
		                  WebviewActivity.startCommonWeb(getContext(), "News", mViewModel.jumpUri);
		              }
		          });
		          addView(mBinding.getRoot());
		      }
		  
		      /**
		       *  和 viewModel 绑定
		       * @param data
		       */
		      @Override
		      public void setData(TitleViewModel data){
		          mBinding.setViewModel(data);
		          mBinding.executePendingBindings();
		          this.mViewModel = data;
		      }
		  }
		  
		  ```
- # 2、和这个view相关的ViewModel，
	- 和JetPackViewModel，的概念不一样。不过MVVM的ViewModel，可以使用JetPack的  viewModel
	- ```java
	  public class BaseCustomViewModel {
	      public String jumpUri;
	      public String title;
	  }
	  public class TitleViewModel extends BaseCustomViewModel {
	  }
	  
	  ```
- # 3、设置给adapter
	- 定义 继承Recyclerview的ViewHolder
		- ```java
		  public class BaseViewHolder extends RecyclerView.ViewHolder {
		      private BaseCustomView itemView;
		      public BaseViewHolder(@NonNull BaseCustomView itemView) {
		          super((View) itemView);
		          this.itemView = itemView;
		      }
		  
		      public void bind(BaseCustomViewModel viewModel){
		          this.itemView.setData(viewModel);
		      }
		  }
		  ```
	- ```java
	  public class NewsListRecyclerViewAdapter extends RecyclerView.Adapter<BaseViewHolder> {
	  
	      private final int VIEW_TYPE_PICTURE_TITLE = 1;
	      private final int VIEW_TYPE_TITLE = 2;
	      private List<BaseCustomViewModel> mItems;
	      private Context mContext;
	  
	      NewsListRecyclerViewAdapter(Context context) {
	          mContext = context;
	      }
	  
	      void setData(List<BaseCustomViewModel> items) {
	          mItems = items;
	          notifyDataSetChanged();
	      }
	  
	      @Override
	      public int getItemCount() {
	          if (mItems != null) {
	              return mItems.size();
	          }
	          return 0;
	      }
	  
	      @Override
	      public int getItemViewType(int position) {
	          if (mItems != null && mItems.get(position) instanceof PictureTitleViewModel) {
	              return VIEW_TYPE_PICTURE_TITLE;
	          }
	          return VIEW_TYPE_TITLE;
	      }
	  
	      @Override
	      public BaseViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
	          if (viewType == VIEW_TYPE_PICTURE_TITLE) {
	              return new BaseViewHolder(new PictureTitleView(parent.getContext()));
	          } else if (viewType == VIEW_TYPE_TITLE) {
	              return new BaseViewHolder(new TitleView(parent.getContext()));
	          }
	          return null;
	      }
	  
	      @Override
	      public void onBindViewHolder(@NonNull BaseViewHolder holder, int position) {
	          holder.bind(mItems.get(position));
	      }
	  }
	  
	  ```