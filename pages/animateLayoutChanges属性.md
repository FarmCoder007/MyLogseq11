- **与上个文件不同这次提到 的可以给每个条目单独进入移除逗带有动画效果**
- ** **
- # 一、在linearlayout等viewgroup上添加一行代码即可实现默认加入移除动画
	- ```java
	  <LinearLayout  
	              android:id="@+id/layoutTransitionGroup"  
	              android:layout_width="match_parent"  
	              android:layout_height="wrap_content"  
	              android:animateLayoutChanges="true"    //就这行
	              android:orientation="vertical"/>  
	  ```
- # 二、可以加入自定义动画的类LayoutTransaction
	- ## 一、使用步骤：
		- **第一步：创建实例**
		- ```java
		  LayoutTransaction transitioner = new LayoutTransition();  
		  ```
	- ## 第二步：创建动画并设置
		- ```java
		  ObjectAnimator animOut = ObjectAnimator.ofFloat(null, "rotation", 0f, 90f, 0f);  
		  transitioner.setAnimator(LayoutTransition.DISAPPEARING, animOut); 
		  ```
		- api
			- ```java
			  public void setAnimator(int transitionType, Animator animator)
			  ```
			- 第一个参数int transitionType：表示当前应用动画的对象范围，取值有：
				- APPEARING —— 元素在容器中出现时所定义的动画。
				- DISAPPEARING —— 元素在容器中消失时所定义的动画。
				- CHANGE_APPEARING —— 由于容器中要显现一个新的元素，其它需要变化的元素所应用的动画
				- CHANGE_DISAPPEARING —— 当容器中某个元素消失，其它需要变化的元素所应用的动画
	- ## 第三部：将LayoutTransaction设置进ViewGroup
		- ```java
		  linearLayout.setLayoutTransition(mTransitioner);
		  ```
		-
			- 第三步中，在API 11之后，所有派生自ViewGroup类，比如LinearLayout，FrameLayout，RelativeLayout等，都具有一个专门用来设置LayoutTransition的方法
				-
-
- ****
- **二代码实现：**