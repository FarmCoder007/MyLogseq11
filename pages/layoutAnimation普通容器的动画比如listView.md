- >**他们只限于第一次初始化的时候才起作用   后期逐个添加不起作用**
- # 一、作用
	- android:layoutAnimation只在viewGroup创建的时候，才会对其中的item添加动画。在创建成功以后，再向其中添加item将不会再有动画。
	- viewGroup控件中添加android:layoutAnimation="@anim/layout_animation"，就可以实现其容器内部控件创建时的动画
- # 二、各属性只有三个字段起作用
  collapsed:: true
	- ```java
	  <layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
	                   android:delay="1"
	                   android:animationOrder="normal"
	                   android:animation="@anim/slide_in_left"/>
	  ```
	- **    ****delay:**指每个Item的动画开始延时，取值是android:animation所指定动画时长的倍数，取值类型可以是float类型，也可以是百分数，默认是0.5;比如我们这里指定的动画是@anim/slide_in_left，而在slide_in_left.xml中指定android:duration=”1000”，即单次动画的时长是1000毫秒，而我们在这里的指定android:delay=”1”，即一个Item的动画会在上一个item动画完成后延时单次动画时长的一倍时间开始，即延时1000毫秒后开始。
	- ** animationOrder:**指viewGroup中的控件动画开始顺序，取值有normal(正序)、reverse(倒序)、random(随机)
	- **animation：**指定每个item入场所要应用的动画。仅能指定res/aim文件夹下的animation定义的动画，不可使用animator动画。
	- 这里最难理解的参数应该是android:delay，它是指viewGroup中各个item开始动画的时间延迟，取值是Item动画时长的倍数。其中item动画是通过android:animation指定的。
- # 三、LayoutAnimation的代码实现——LayoutAnimationController【标签对应的类】
	- **2个构造函数：**
	  collapsed:: true
		- ```java
		  public LayoutAnimationController(Animation animation)
		  public LayoutAnimationController(Animation animation, float delay)
		  ```
	- **常用函数**
	  collapsed:: true
		- ```java
		  /**
		   * 设置animation动画
		   */
		  public void setAnimation(Animation animation)
		  /**
		   * 设置单个item开始动画延时
		   */
		  public void setDelay(float delay)
		  /**
		   * 设置viewGroup中控件开始动画顺序，取值为ORDER_NORMAL、ORDER_REVERSE、ORDER_RANDOM
		   */
		  public void setOrder(int order)
		  ```
	- **使用：**
		- ```java
		  //代码设置通过加载XML动画设置文件来创建一个Animation对象；
		          Animation animation= AnimationUtils.loadAnimation(this,R.anim.slide_in_left);   //得到一个LayoutAnimationController对象；
		          LayoutAnimationController controller = new LayoutAnimationController(animation);   //设置控件显示的顺序；
		          controller.setOrder(LayoutAnimationController.ORDER_REVERSE);   //设置控件显示间隔时间；
		          controller.setDelay(0.3f);   //为ListView设置LayoutAnimationController属性；
		          mListView.setLayoutAnimation(controller);
		          mListView.startLayoutAnimation();
		  ```
- # 四、**GridLayoutAnimation的XML实现——gridLayoutAnimation**
	- ## 一、常用标签
	  collapsed:: true
		- ```xml
		  <gridLayoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
		                       android:rowDelay="75%"
		                       android:columnDelay="60%"
		                       android:directionPriority="none"
		                       android:direction="bottom_to_top|right_to_left"
		                       android:animation="@android:anim/slide_in_left"/>
		  ```
		- rowDelay:每一行动画开始的延迟。与LayoutAnimation一样，可以取百分数，也可以取浮点数。取值意义为，当前android:animation所指动画时长的倍数。
		- columnDelay：每一列动画开始的延迟。取值类型及意义与rowDelay相同。
		- directionPriority：方向优先级。取值为row,collumn,none，意义分别为：行优先，列优先，和无优先级（同时进行）;具体意义，后面会细讲
		- **direction：**gridview动画方向。
		  取值有四个：left_to_right：列，从左向右开始动画
		  right_to_left ：列，从右向左开始动画
		  top_to_bottom：行，从上向下开始动画
		  bottom_to_top：行，从下向上开始动画
		  这四个值之间可以通过“|”连接，从而可以取多个值。很显然left_to_right和right_to_left是互斥的，top_to_bottom和bottom_to_top是互斥的。如果不指定 direction字段，默认值为left_to_right | top_to_bottom；即从上往下，从左往右。
		- animation: gridview内部元素所使用的动画。
		  作用：第一：gridview中各个元素的出场顺序为从上往下，从左往右。 
		  第二：gridLayoutAnimation仅在gridview第一次创建时各个元素才会有出场动画，在创建成功以后，再向其中添加数据就不会再有动画。这一点与layoutAnimation相同。
	- ## 二、Xml实现gridVIew条目动画
	  collapsed:: true
		- ```xml
		  <?xml version="1.0" encoding="utf-8"?>
		  <gridLayoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
		                       android:rowDelay="75%"
		                       android:columnDelay="60%"
		                       android:directionPriority="none"
		                       android:animation="@anim/slide_in_left"/>
		  ```
		- 里边的每个条目进来的动画为：anim/slide_in_left
			- ```java
			  <set xmlns:android="http://schemas.android.com/apk/res/android" android:duration="1000">
			      <translate android:fromXDelta="-50%p" android:toXDelta="0"/>
			      <alpha android:fromAlpha="0.0" android:toAlpha="1.0" />
			  </set>
			  ```
		- **然后在布局中应用到gridview中**
			- ```java
			  <GridView
			              android:id="@+id/grid"
			              android:layout_width="match_parent"
			              android:layout_height="match_parent"
			              android:columnWidth="60dp"
			              android:gravity="center"
			              android:horizontalSpacing="10dp"
			              android:layoutAnimation="@anim/gride_animation"
			              android:numColumns="auto_fit"
			              android:stretchMode="columnWidth"
			              android:verticalSpacing="10dp"/>
			  ```
		- >**注意：****gridLayoutAnimation与layoutAnimation一样，都只是在viewGroup创建的时候，会对其中的item添加进入动画，在创建完成后，再添加数据将不会再有动画！**
		- #
		-
	- # 三、其他标签字段的详解
	  collapsed:: true
		- directionPriority指gridview动画优先级，取值有row,column,none.意义分别为行优先，列优先，和无优先级（同时进行）。
		  direction表示gridview的各个item的动画方向，取值如下，可以通过“|”连接多个属性值。
		  取值有四个：
		- left_to_right：列，从左向右开始动画
		- right_to_left ：列，从右向左开始动画
		- top_to_bottom：行，从上向下开始动画
		- bottom_to_top：行，从下向上开始动画
	- ## 四、gridLayoutAnimation对应的代码实现对应类为GridLayoutAnimationController
		- **1、构造函数**
		  collapsed:: true
			- ```java
			  public GridLayoutAnimationController(Animation animation)
			  public GridLayoutAnimationController(Animation animation, float columnDelay, float rowDelay)
			  其中animation对应标签属性中的android:animation
			  columnDelay对应标签属性中的android:columnDelay，取值为float类型
			  rowDelay对应标签属性中的android:rowDelay，取值为float类型
			  ```
		- **2、常用函数**
		  collapsed:: true
			- ```java
			  /**
			   * 设置列动画开始延迟
			   */
			  public void setColumnDelay(float columnDelay)
			  /**
			   * 设置行动画开始延迟
			   */
			   public void setRowDelay(float rowDelay)
			   /**
			   * 设置gridview动画的入场方向。取值有：DIRECTION_BOTTOM_TO_TOP、DIRECTION_TOP_TO_BOTTOM、DIRECTION_LEFT_TO_RIGHT、DIRECTION_RIGHT_TO_LEFT
			   */
			   public void setDirection(int direction)
			   /**
			   * 动画开始优先级，取值有PRIORITY_COLUMN、PRIORITY_NONE、PRIORITY_ROW
			   */
			   public void setDirectionPriority(int directionPriority)
			  ```
		- 3、使用给容器添加动画无论是xml还是代码实现的   条目动画的xml都要有的
			- ```xml
			  <?xml version="1.0" encoding="utf-8"?>
			  <set xmlns:android="http://schemas.android.com/apk/res/android" android:duration="1000">
			      <translate android:fromXDelta="-50%p" android:toXDelta="0"/>
			      <alpha android:fromAlpha="0.0" android:toAlpha="1.0" />
			  </set>
			  ```
			- ```java
			  Animation animation = AnimationUtils.loadAnimation(MyActivity.this,R.anim.slide_in_left);
			          GridLayoutAnimationController controller = new GridLayoutAnimationController(animation);
			          controller.setColumnDelay(0.75f);
			          controller.setRowDelay(0.5f);
			          controller.setDirection(GridLayoutAnimationController.DIRECTION_BOTTOM_TO_TOP|GridLayoutAnimationController.DIRECTION_LEFT_TO_RIGHT);
			          controller.setDirectionPriority(GridLayoutAnimationController.PRIORITY_NONE);
			          grid.setLayoutAnimation(controller);
			          grid.startLayoutAnimation();
			  ```
-