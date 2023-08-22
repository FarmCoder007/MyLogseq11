# Adapter
	- ## 旧版Adapter的编写,ViewHolder也放里
	  collapsed:: true
		- ```java
		  public class NewsListRecyclerViewAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {
		  
		      private final int VIEW_TYPE_PICTURE_TITLE = 1;
		      private final int VIEW_TYPE_TITLE = 2;
		      private List<NewsListBean.Contentlist> mItems;
		      private Context mContext;
		  
		      NewsListRecyclerViewAdapter(Context context) {
		          mContext = context;
		      }
		  
		      void setData(List<NewsListBean.Contentlist> items) {
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
		          if (mItems != null && mItems.get(position).imageurls != null && mItems.get(position).imageurls.size() > 1) {
		              return VIEW_TYPE_PICTURE_TITLE;
		          }
		          return VIEW_TYPE_TITLE;
		      }
		  
		      @Override
		      public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
		          View view;
		          if (viewType == VIEW_TYPE_PICTURE_TITLE) {
		              view = LayoutInflater.from(mContext).inflate(R.layout.picture_title_view, parent, false);
		              return new PictureTitleViewHolder(view);
		          } else if (viewType == VIEW_TYPE_TITLE) {
		              view = LayoutInflater.from(mContext).inflate(R.layout.title_view, parent, false);
		              return new TitleViewHolder(view);
		          }
		  
		          return null;
		      }
		  
		      private class PictureTitleViewHolder extends RecyclerView.ViewHolder {
		          public TextView titleTextView;
		          public AppCompatImageView picutureImageView;
		  
		          public PictureTitleViewHolder(@NonNull View itemView) {
		              super(itemView);
		              titleTextView = itemView.findViewById(R.id.item_title);
		              picutureImageView = itemView.findViewById(R.id.item_image);
		              itemView.setOnClickListener(new View.OnClickListener() {
		                  @Override
		                  public void onClick(View v) {
		                      WebviewActivity.startCommonWeb(mContext, "News", v.getTag()+"");
		                  }
		              });
		          }
		      }
		  
		  
		      private class TitleViewHolder extends RecyclerView.ViewHolder {
		          public TextView titleTextView;
		  
		          public TitleViewHolder(@NonNull View itemView) {
		              super(itemView);
		              titleTextView = itemView.findViewById(R.id.item_title);
		              itemView.setOnClickListener(new View.OnClickListener() {
		                  @Override
		                  public void onClick(View v) {
		                      WebviewActivity.startCommonWeb(mContext, "News", v.getTag()+"");
		                  }
		              });
		          }
		      }
		      @Override
		      public void onBindViewHolder(@NonNull RecyclerView.ViewHolder holder, int position) {
		          holder.itemView.setTag(mItems.get(position).link);
		          if(holder instanceof PictureTitleViewHolder){
		              ((PictureTitleViewHolder) holder).titleTextView.setText(mItems.get(position).title);
		              Glide.with(holder.itemView.getContext())
		                      .load(mItems.get(position).imageurls.get(0).url)
		                      .transition(withCrossFade())
		                      .into(((PictureTitleViewHolder) holder).picutureImageView);
		          } else if(holder instanceof TitleViewHolder) {
		              ((TitleViewHolder) holder).titleTextView.setText(mItems.get(position).title);
		          }
		      }
		  }
		  
		  ```
	- ## MVVM中思想
		- 1、ViewHodler单独类抽离出来
		- 2、里边的itemView。利用自定义布局(自定义view)，与这个view相关操作都放里，单一职责原则。哪里用到，哪里用，修改东西只需要修改这个自定义view类
			- 通过databinding。自动绑定
		- [[MVVM-adapter实战,解耦]]
- # 将base基类的东西分模块化
	- ## baseView的
	  collapsed:: true
		- ## base
			- BaseCustomView:自定实现数据绑定
			  collapsed:: true
				- ```java
				  public abstract class BaseCustomView<VIEW extends ViewDataBinding, DATA extends BaseCustomViewModel> extends LinearLayout implements IBaseCustomView<DATA>{
				      protected VIEW binding;
				      protected DATA data;
				  
				      public BaseCustomView(Context context) {
				          super(context);
				          init();
				      }
				  
				      public BaseCustomView(Context context, @Nullable AttributeSet attrs) {
				          super(context, attrs);
				          init();
				      }
				  
				      public BaseCustomView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
				          super(context, attrs, defStyleAttr);
				          init();
				      }
				  
				      public BaseCustomView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
				          super(context, attrs, defStyleAttr, defStyleRes);
				          init();
				      }
				  
				      public void init(){
				          LayoutInflater inflater = (LayoutInflater) getContext().getSystemService(Context.LAYOUT_INFLATER_SERVICE);
				          binding = DataBindingUtil.inflate(inflater, getLayoutId(), this, false);
				          binding.getRoot().setOnClickListener(new OnClickListener() {
				              @Override
				              public void onClick(View v) {
				                  onRootClicked(v);
				              }
				          });
				          addView(binding.getRoot());
				      }
				  
				      public abstract int getLayoutId();
				  
				      public abstract void onRootClicked(View view);
				  
				      @Override
				      public void setData(DATA data){
				          setDataToView(data);
				          binding.executePendingBindings();
				          this.data = data;
				      }
				  
				      protected abstract void setDataToView(DATA data);
				  }
				  
				  ```
		- ## 使用,TitleViewBinding 为xml里使用了databinding生成的
		  collapsed:: true
			- ```java
			  public class TitleView extends BaseCustomView<TitleViewBinding, TitleViewModel> {
			      public TitleView(Context context) {
			          super(context);
			      }
			  
			      @Override
			      public int getLayoutId() {
			          return R.layout.title_view;
			      }
			  
			      @Override
			      public void onRootClicked(View view) {
			          WebviewActivity.startCommonWeb(getContext(), "News", data.jumpUri);
			      }
			  
			      @Override
			      protected void setDataToView(TitleViewModel titleViewModel) {
			          binding.setViewModel(data);
			      }
			  }
			  ```
	- ## ViewModel的
	  collapsed:: true
		- base
			- ```java
			  public class BaseCustomViewModel {
			      public String jumpUri;
			      public String title;
			  }
			  ```
		- 使用
			- ```java
			  public class TitleViewModel extends BaseCustomViewModel {
			  }
			  
			  ```