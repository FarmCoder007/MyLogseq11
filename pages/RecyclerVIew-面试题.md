## 1、[[RecyclerView缓存机制-面试]]
- ## 2、[[自定义ItemDecoration实现吸顶-面试]]
- ## 3、ConcatAdapter#card
	- 作用：可以合并不同adapter，管理各自的数据源
	- ## 1、怎么合并多个Adapter的
		- ConcatAdapterController是操作多个Adapter聚合到一个RecyclerView的控制类
		- 通过增加一个Controller。重写onCreateViewHolder、onBindViewHolder和getItemCount等方法
	- ## 2、itemType重复怎么办
		- 根据子Adapter的viewType，递增生成唯一的globalType并且存储着对应的映射关系