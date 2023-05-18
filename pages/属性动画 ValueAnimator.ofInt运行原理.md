title:: 属性动画 ValueAnimator.ofInt运行原理

	-
- # 前言
  collapsed:: true
	- ```
	  val valueAnimator = ValueAnimator.ofInt(0, 100,300).apply {
	              duration = 1000
	              interpolator = LinearInterpolator()
	              addUpdateListener {
	              }
	          }
	   valueAnimator.start()
	  ```
	- 问题：
	- 0-100 中经过的时间到底是0.5s还是0.33s？
	  ValueAnimator.ofInt(0, 300)有区别吗？
- # 源码解析
	- 那我们先从ValueAnimator.ofInt()开始分析
	  collapsed:: true
		- ```
		  //ValueAnimator
		  public static ValueAnimator ofInt(int... values) {
		          ValueAnimator anim = new ValueAnimator();
		          anim.setIntValues(values);
		          return anim;
		  }
		  ```
	- 我们重点是要查看values的去向,进入anim.setFloatValues(value)方法
	  collapsed:: true
		- ```
		  //ValueAnimator
		  public void setIntValues(int... values) {
		     if (values == null || values.length == 0) {
		          return;
		      }
		      if (mValues == null || mValues.length == 0) {
		          setValues(PropertyValuesHolder.ofInt("", values));
		      } else {
		          PropertyValuesHolder valuesHolder = mValues[0];
		          valuesHolder.setIntValues(values);
		      }
		      // New property/values/target should cause re-initialization prior to starting
		      mInitialized = false;
		  } 
		  ```
	- mValues为null，执行setValues(PropertyValuesHolder.ofInt("", values));最终调用了IntPropertyValuesHolder的构造方法
		- ```
		  //IntPropertyValuesHolder
		  public IntPropertyValuesHolder(String propertyName, int... values) {
		         super(propertyName);
		         setIntValues(values);
		  }
		  ```
		- ```
		  //IntPropertyValuesHolder
		  public void setIntValues(int... values) {
		        mValueType = int.class;
		        mKeyframes = KeyframeSet.ofInt(values);
		    }
		  ```