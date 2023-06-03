- # 一、概念
	- **Key****Frame****作用控制动画速率   **该类包含了一个animation类的time/value值对。
- # 二、构造函数
	- ```java
	  ofFloat(float fraction, float value)
	  ofFloat(float fraction)
	  ofInt(float fraction)
	  ofInt(float fraction, int value)
	  ofObject(float fraction)
	  ofObject(float fraction, Object value)
	  ```
	- 参数一fraction是表示当前的显示进度，即从加速器【插值器】中getInterpolation()函数的返回值；
	- 参数二value 表示当前应该所在的位置
	- **比如**Keyframe.ofFloat(0, 0)表示动画进度为0时，动画所在的数值位置为0；Keyframe.ofFloat(0.25f, -20f)表示动画进度为25%时，动画所在的数值位置为-20；Keyframe.ofFloat(1f,0)表示动画结束时，动画所在的数值位置为0；
- # 三、PropertyValuesHolder使用KeyFrame对象  设置动画的速率
  collapsed:: true
	- ```java
	  public static PropertyValuesHolder ofKeyframe(String propertyName, Keyframe... values)
	  ```
		- **propertyName**：动画所要操作的属性名
		- **values：Keyframe的列表**，PropertyValuesHolder会根据每个Keyframe的设定，定时将指定的值输出给动画。
	- ## 实例代码：
		- 第一步：生成Keyframe对象；
		- 第二步：利用PropertyValuesHolder.ofKeyframe()生成PropertyValuesHolder对象
		- 第三步：ObjectAnimator.ofPropertyValuesHolder()生成对应的Animator
		- ```java
		  Keyframe frame0 = Keyframe.ofFloat(0f, 0);  
		      Keyframe frame1 = Keyframe.ofFloat(0.1f, -20f);  
		      Keyframe frame2 = Keyframe.ofFloat(1, 0);  
		      PropertyValuesHolder frameHolder = PropertyValuesHolder.ofKeyframe("rotation",frame0,frame1,frame2);  
		      Animator animator = ObjectAnimator.ofPropertyValuesHolder(mImage,frameHolder);  
		      animator.setDuration(1000);  
		  animator.start();  
		  ```
- # 四、Keyframe常用的其他函数
  collapsed:: true
	- ```java
	   /** 
	       * 设置fraction参数，即Keyframe所对应的进度 
	       */  
	      public void setFraction(float fraction)   
	      /** 
	       * 设置当前Keyframe所对应的值 
	       */  
	      public void setValue(Object value)  
	      /** 
	       * 设置Keyframe动作期间所对应的插值器 
	       */  
	  public void setInterpolator(TimeInterpolator interpolator)  
	  其中keyFrame设置插值器时  那么keyFrame是无效的
	  Keyframe frame0 = Keyframe.ofFloat(0f, 0);  
	  Keyframe frame1 = Keyframe.ofFloat(0.1f, -20f);  
	  frame1.setInterpolator(new BounceInterpolator());  
	  Keyframe frame2 = Keyframe.ofFloat(1f, 20f);  
	  frame2.setInterpolator(new LinearInterpolator());
	  在上面的代码中，我们给frame1设置了插值器BounceInterpolator，那么在frame0到frame1的中间值计算过程中，就是用的就是回弹插值器；
	  同样，我们给frame2设置了线性插值器（LinearInterpolator），所以在frame1到frame2的中间值计算过程中，使用的就是线性插值器
	  很显然，给Keyframe.ofFloat(0f, 0)设置插值器是无效的，因为它是第一帧
	  使用ofFloat(float fraction)只会有个当前进度  设置value有单独的方法
	  Keyframe frame2 = Keyframe.ofFloat(1);  
	  frame2.setValue(0f);
	  对于Keyframe而言，fraction和value这两个参数是必须有的，所以无论用哪种方式实例化Keyframe都必须保证这两个值必须被初始化。
	  这里没有设置插值器，会使用默认的线性插值器（LinearInterpolator）
	  ```
- # 五、keyFrame.ofObject方法时
	- 凡是使用ofObject来做动画的时候，都必须调用frameHolder.setEvaluator显示设置Evaluator，因为系统根本是无法知道，你动画的中间值Object真正是什么类型的。
- # 六、结论
	- **使用Key****Frame****时没有设置第一帧 **(0f, 0);  进度0值为0时默认从最近一帧开始
	- **使用Key****Frame****时没有设置最后一帧 **(1, 0);  进度1值为0时默认从最后一帧结束
	- 如果去掉第0帧，将以第一个关键帧为起始位置
	- 如果去掉结束帧，将以最后一个关键帧为结束位置
	- 使用Keyframe来构建动画，至少要有两个或两个以上帧
- # 七、多动画实现使用ofPropertyValuesHolder   创建多个PropertyValuesHolder
	- ```java
	   /** 
	       * 左右震动效果 
	       */  
	      Keyframe frame0 = Keyframe.ofFloat(0f, 0);  
	      Keyframe frame1 = Keyframe.ofFloat(0.1f, -20f);  
	      Keyframe frame2 = Keyframe.ofFloat(0.2f, 20f);  
	      Keyframe frame3 = Keyframe.ofFloat(0.3f, -20f);  
	      Keyframe frame4 = Keyframe.ofFloat(0.4f, 20f);  
	      Keyframe frame5 = Keyframe.ofFloat(0.5f, -20f);  
	      Keyframe frame6 = Keyframe.ofFloat(0.6f, 20f);  
	      Keyframe frame7 = Keyframe.ofFloat(0.7f, -20f);  
	      Keyframe frame8 = Keyframe.ofFloat(0.8f, 20f);  
	      Keyframe frame9 = Keyframe.ofFloat(0.9f, -20f);  
	      Keyframe frame10 = Keyframe.ofFloat(1, 0);  
	      PropertyValuesHolder frameHolder1 = PropertyValuesHolder.ofKeyframe("rotation", frame0, frame1, frame2, frame3, frame4,frame5, frame6, frame7, frame8, frame9, frame10);  
	        
	        
	      /** 
	       * scaleX放大1.1倍 
	       */  
	      Keyframe scaleXframe0 = Keyframe.ofFloat(0f, 1);  
	      Keyframe scaleXframe1 = Keyframe.ofFloat(0.1f, 1.1f);  
	      Keyframe scaleXframe2 = Keyframe.ofFloat(0.2f, 1.1f);  
	      Keyframe scaleXframe3 = Keyframe.ofFloat(0.3f, 1.1f);  
	      Keyframe scaleXframe4 = Keyframe.ofFloat(0.4f, 1.1f);  
	      Keyframe scaleXframe5 = Keyframe.ofFloat(0.5f, 1.1f);  
	      Keyframe scaleXframe6 = Keyframe.ofFloat(0.6f, 1.1f);  
	      Keyframe scaleXframe7 = Keyframe.ofFloat(0.7f, 1.1f);  
	      Keyframe scaleXframe8 = Keyframe.ofFloat(0.8f, 1.1f);  
	      Keyframe scaleXframe9 = Keyframe.ofFloat(0.9f, 1.1f);  
	      Keyframe scaleXframe10 = Keyframe.ofFloat(1, 1);  
	      PropertyValuesHolder frameHolder2 = PropertyValuesHolder.ofKeyframe("ScaleX",scaleXframe0,scaleXframe1,scaleXframe2,scaleXframe3,scaleXframe4,scaleXframe5,scaleXframe6,scaleXframe7,scaleXframe8,scaleXframe9,scaleXframe10);  
	        
	        
	      /** 
	       * scaleY放大1.1倍 
	       */  
	      Keyframe scaleYframe0 = Keyframe.ofFloat(0f, 1);  
	      Keyframe scaleYframe1 = Keyframe.ofFloat(0.1f, 1.1f);  
	      Keyframe scaleYframe2 = Keyframe.ofFloat(0.2f, 1.1f);  
	      Keyframe scaleYframe3 = Keyframe.ofFloat(0.3f, 1.1f);  
	      Keyframe scaleYframe4 = Keyframe.ofFloat(0.4f, 1.1f);  
	      Keyframe scaleYframe5 = Keyframe.ofFloat(0.5f, 1.1f);  
	      Keyframe scaleYframe6 = Keyframe.ofFloat(0.6f, 1.1f);  
	      Keyframe scaleYframe7 = Keyframe.ofFloat(0.7f, 1.1f);  
	      Keyframe scaleYframe8 = Keyframe.ofFloat(0.8f, 1.1f);  
	      Keyframe scaleYframe9 = Keyframe.ofFloat(0.9f, 1.1f);  
	      Keyframe scaleYframe10 = Keyframe.ofFloat(1, 1);  
	      PropertyValuesHolder frameHolder3 = PropertyValuesHolder.ofKeyframe("ScaleY",scaleYframe0,scaleYframe1,scaleYframe2,scaleYframe3,scaleYframe4,scaleYframe5,scaleYframe6,scaleYframe7,scaleYframe8,scaleYframe9,scaleYframe10);  
	        
	      /** 
	       * 构建动画 
	       */  
	      Animator animator = ObjectAnimator.ofPropertyValuesHolder(mImage, frameHolder1,frameHolder2,frameHolder3);  
	      animator.setDuration(1000);  
	      animator.start();  
	  ```