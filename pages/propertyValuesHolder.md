# 一、构造,ValueAnimator和ObjectAnimator
	- 【ValueAnimator和ObjectAnimator】的api中除了ofInt(),ofFloat(),ofObject()函数还有另外一个构造ofPropertyValuesHolder
	- ```java
	    /** 
	     * valueAnimator的 
	     */  
	   public static ValueAnimator ofPropertyValuesHolder(PropertyValuesHolder... values)   
	    /** 
	     * ObjectAnimator的 
	     */  
	   public static ObjectAnimator ofPropertyValuesHolder(Object target,PropertyValuesHolder... values)
	  ```
- # 二、ofPropertyValuesHolder的四个构造函数
	- ```java
	  public static PropertyValuesHolder ofFloat(String propertyName, float... values)  
	  public static PropertyValuesHolder ofInt(String propertyName, int... values)   
	  public static PropertyValuesHolder ofObject(String propertyName, TypeEvaluator evaluator,Object... values)  
	  public static PropertyValuesHolder ofKeyframe(String propertyName, Keyframe... values)
	  
	  ```
	- 参数一：要操做的属性名字  和objectAnimator操作的属性名字一样
	  id:: 647b0c32-ea0c-4fa8-a5bc-7b1c75b4e658
	  参数二 属性对应的一系列值，可变长度，如果只指定了一个，那么ObjectAnimator会通过查找getProperty()方法来获得初始值。不理解的同学请参看
- # 三、ObjectAnimator设置ofPropertyValuesHolder方法
	- ```java
	  public static ObjectAnimator ofPropertyValuesHolder(Object target,PropertyValuesHolder... values) 
	  ```
	- target：指需要执行动画的控件
	- values：是一个可变长参数，可以传进去多个PropertyValuesHolder实例，由于每个PropertyValuesHolder实例都会针对一个属性做动画，所以如果传进去多个PropertyValuesHolder实例，将会对控件的多个属性同时做动画操作。
	- ## 使用示例1：**使用**PropertyValuesHolder.ofFloat和ofInt只需要传入属性名字 和一系列值
		- ```java
		  PropertyValuesHolder rotationHolder = PropertyValuesHolder.ofFloat("Rotation", 60f, -60f, 40f, -40f, -20f, 20f, 10f, -10f, 0f);  
		  PropertyValuesHolder colorHolder = PropertyValuesHolder.ofInt("BackgroundColor", 0xffffffff, 0xffff00ff, 0xffffff00, 0xffffffff);  
		  ObjectAnimator animator = ObjectAnimator.ofPropertyValuesHolder(mTextView, rotationHolder, colorHolder);  
		  animator.setDuration(3000);  
		  animator.setInterpolator(new AccelerateInterpolator());  
		  animator.start();
		  ```
	- ## 使用示例2：PropertyValuesHolder.ofObject创建PropertyValuesHolder对象
		- 构造函数
			- ```java
			  public static PropertyValuesHolder ofObject(String propertyName, TypeEvaluator evaluator,Object... values) 
			  ```
		- propertyName:ObjectAnimator动画操作的属性名;
		- evaluator:Evaluator实例，Evaluator是将当前动画进度计算出当前值的类，可以使用系统自带的IntEvaluator、FloatEvaluator也可以自定义，
		- values：可变长参数，表示操作动画属性的值
- # 四、其他函数
	- ```java
	  /** 
	   * 设置动画的Evaluator 
	   */  
	  public void setEvaluator(TypeEvaluator evaluator)  
	  /** 
	   * 用于设置ofFloat所对应的动画值列表 
	   */  
	  public void setFloatValues(float... values)  
	  /** 
	   * 用于设置ofInt所对应的动画值列表 
	   */  
	  public void setIntValues(int... values)  
	  /** 
	   * 用于设置ofKeyframe所对应的动画值列表 
	   */  
	  public void setKeyframes(Keyframe... values)  
	  /** 
	   * 用于设置ofObject所对应的动画值列表 
	   */  
	  public void setObjectValues(Object... values)  
	  /** 
	   * 设置动画属性名 
	   */  
	  public void setPropertyName(String propertyName)
	  ```