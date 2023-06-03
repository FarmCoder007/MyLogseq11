- # 一、概念
	- 此类将一组Animator对象按指定的顺序播放。
- # 二、常用方法一playSequentially
  collapsed:: true
	- ```java
	  public void playSequentially(Animator... items);【常用】
	  public void playSequentially(List<Animator> items);
	  ```
	- 两个方法都是传入多个Animator动画对象的
	- 创建多个ObjectAnimator对象  使用animatorSet按顺序播放
	- ```java
	    ObjectAnimator tv1BgAnimator = ObjectAnimator.ofInt(mTv1, "BackgroundColor",  0xffff00ff, 0xffffff00, 0xffff00ff);
	      ObjectAnimator tv1TranslateY = ObjectAnimator.ofFloat(mTv1, "translationY", 0, 300, 0);
	      ObjectAnimator tv2TranslateY = ObjectAnimator.ofFloat(mTv2, "translationY", 0, 400, 0);
	  
	      AnimatorSet animatorSet = new AnimatorSet();
	      animatorSet.playSequentially(tv1BgAnimator,tv1TranslateY,tv2TranslateY);
	      animatorSet.setDuration(1000);
	  animatorSet.start();
	  ```
- # 三、常用方法二playTogether多个动画一起执行
	- ```java
	  ObjectAnimator tv1BgAnimator = ObjectAnimator.ofInt(mTv1, "BackgroundColor",  0xffff00ff, 0xffffff00, 0xffff00ff);
	  ObjectAnimator tv1TranslateY = ObjectAnimator.ofFloat(mTv1, "translationY", 0, 400, 0);
	  ObjectAnimator tv2TranslateY = ObjectAnimator.ofFloat(mTv2, "translationY", 0, 400, 0);
	  
	  AnimatorSet animatorSet = new AnimatorSet();
	  animatorSet.playTogether(tv1BgAnimator,tv1TranslateY,tv2TranslateY);
	  animatorSet.setDuration(1000);
	  animatorSet.start();
	  ```
- # 四、playTogether  playSequentially区别
	- **playTogether**
		- 只是一个时间点上的一起开始，对于开始后，各个动画怎么操作就是他们自己的事了，至于各个动画结不结束也是他们自已的事了。
	- **playSequentially**** **
		- 就是把激活一个动画之后，动画之后的操作就是动画自己来负责了，这个动画结束之后，再激活下一个动画。如果上一个动画没有结束，那下一个动画就永远也不会被激活。
- # 五、总结实现无限循环的组合动画
	- 只需要把每个动画设置成
	- ```java
	  setRepeatCount(ValueAnimator.INFINITE);无限循环即可  然后使用playTogether
	  ```
- # 六、自由设置动画顺序AnimatorSet.Builder
	- //调用AnimatorSet中的play方法是获取AnimatorSet.Builder对象的唯一途径
	- //表示要播放哪个动画
	- public Builder play(Animator anim)
- # 七、AnimatorSet.Builder的一些常见函数用于控制动画的顺序
  collapsed:: true
	- ```java
	  //和前面动画一起执行
	  public Builder with(Animator anim)
	  //执行前面的动画后才执行该动画
	  public Builder before(Animator anim)
	  //执行先执行输入这个动画再执行前面动画【意思是前边的在此之后执行】
	  public Builder after(Animator anim)
	  //延迟n毫秒之后执行动画
	  public Builder after(long delay)
	  ```
	- **play(Animator anim)表示当前在播放哪个动画**，另外的with(Animator anim)、before(Animator anim)、after(Animator anim)都是以**play中的当前所**播放的动画为基准的。
	- 比如，当play(playAnim)与before(beforeAnim)共用，则表示在播放beforeAnim之前，先播放playAnim动画；同样，当play(playAnim)与after(afterAnim)共用时，则表示在在播放afterAnim动画之后，再播放playAnim动画。
- # 八、AnimatorSet的监听函数
	- **他和**valueAnimator的监听函数一样
	- **添加方法**
		- ```java
		  animatorSet.addListener(new Animator.AnimatorListener()
		  ```
	- 注意：
		- 1 AnimatorSet的监听函数也只是用来监听AnimatorSet的状态的，与其中的动画无关；
		- 2 AnimatorSet中没有设置循环的函数，所以AnimatorSet监听器中永远无法运行到onAnimationRepeat()中！
- # 九、objectAnimator和animator的通用方法有什么区别；
	- 部分通用方法在单个设置和animatorset上设置的区别
		- ```java
		  //设置单次动画时长
		  public AnimatorSet setDuration(long duration);
		  //设置加速器
		  public void setInterpolator(TimeInterpolator interpolator)
		  //设置ObjectAnimator动画目标控件
		  public void setTarget(Object target)
		  ```
	- 大部分通用方法除了setStartDelay这方法区别就是：
		- **1**在AnimatorSet中设置以后，会覆盖单个ObjectAnimator中的设置；即如果AnimatorSet中没有设置，那么就以ObjectAnimator中的设置为准。如果AnimatorSet中设置以后，ObjectAnimator中的设置就会无效。
		- **2** AnimatorSet的setTartget函数设置了目标控件，那么单个动画中的目标控件都以AnimatorSet设置的为准
	- **单独方法的不同****setStartDelay****：**唯一的一个方法在AnimatorSet设置之后不对里边单个的ObjectAnimator起作用的
		- //设置延时开始动画时长
		- public void setStartDelay(long startDelay)
	- **作用总结：**
		- **1**setStartDelay函数不会覆盖单个动画的延时，而且仅针对性的延长AnimatorSet的激活时间，单个动画的所设置的setStartDelay仍对单个动画起作用。AnimatorSet的延时是仅针对性的延长AnimatorSet激活时间的，对单个动画的延时设置没有影响。
		- **2**AnimatorSet真正激活延时 = AnimatorSet.startDelay+第一个动画.startDelay
		- **3**在AnimatorSet激活之后，第一个动画绝对是会开始运行的，后面的动画则根据自己是否延时自行处理。