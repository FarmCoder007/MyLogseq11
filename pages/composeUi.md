- ## 使用Compose写UI
  collapsed:: true
	- compose是android新推出的UI工具包，使用可组合函数以声明式来构建UI，不再使用xml布局文件
	- 使用compose构建ui的例子：
	  collapsed:: true
		- ![image.png](../assets/image_1684391017018_0.png)
		-
	- 运行效果图
	  collapsed:: true
		- ![image.png](../assets/image_1684391037897_0.png)
	- 既然Compose是全新的一套UI工具包，那么它是如何构建UI的，是否用到了诸如LinearLayout、RelativeLayout、Button、Text这些android view呢？
	- 让我们来分析上图的布局
	- 遍历view，输入如下：
	  collapsed:: true
		- ![image.png](../assets/image_1684391051742_0.png)
	- 没有ImageView、TextView等android view，看来compose是完全自己实现了一套布局和绘制流程。
	- 虽说compose没有使用android的ui组件，但是如果没有硬件加速，还是会有很多子view，例如关闭compose的一个标志位，会出现很多view。如反射设置androidx.compose.ui.platform.AndroidComposeView.isRenderNodeCompatible为false，再打印view树如图：
	  collapsed:: true
		- ![image.png](../assets/image_1684391066730_0.png)
	- 多出了ViewLayerContainer，它包含了很多ViewLayer。后面再分析为什么会出现这么多ViewLayer。
- ## 可重组方法执行流程
  collapsed:: true
	- Column方法，最后执行Layout方法
	  collapsed:: true
		- ![image.png](../assets/image_1684391338562_0.png)
	- Image方法，最后执行Layout方法
	  collapsed:: true
		- ![image.png](../assets/image_1684391352377_0.png)
	- Text方法一样最后执行Layout方法：Layout方法
	  collapsed:: true
		- ![image.png](../assets/image_1684391368858_0.png)
	- 执行ComposeUiNode.Constructor会创建一个LayoutNode
	  collapsed:: true
		- ![image.png](../assets/image_1684391380281_0.png)
		- ![image.png](../assets/image_1684391388575_0.png)
	- 最后所有的方法都执行到了ComposeNode方法，这里边进行创建LayoutNode并创建子LayoutNode。基本上可以确定compose ui由LayoutNode来组成
- ## compose ui 和android ui嵌套用法
  collapsed:: true
	- 在androidview嵌入compose ui
		- Actvity根布局：在activity调用setContent {}方法
		- 其他：在其他view嵌入ComposeView，然后调用ComposeView的setContent{}方法
	- 在compose ui嵌入android view
		- 在可重组函数内部调用AndroidView可重组函数来嵌入android view
- ## compose ui结构图
  collapsed:: true
	- 1、android ui和compose ui结构图
	  collapsed:: true
		- ![image.png](../assets/image_1684392051843_0.png){:height 716, :width 716}
	- 2、LayoutNode的LayoutNodeWrapper和Modifer链
	  collapsed:: true
		- ![image.png](../assets/image_1684392069097_0.png)
- ## 绘制流程
  collapsed:: true
	- compose的ui绘制不依赖android ui控件，是完全实现了一套绘制流程。通过LayoutNode树完成绘制，LayoutNode内部通过LayoutNodeWrapper来层层绘制，每层最终由DrawModifier(Modifier子类）来进行绘制
		- 1、绘制方法传递：从android层传递到LayoutNode，LayoutNode内部绘制结束之后继续绘制子LayoutNode
		  collapsed:: true
			- compose从AndroidComposeView的dispatchDraw开始绘制，调用root: LayoutNode的draw方法
			- LayoutNode的draw方法调用outerLayoutNodeWrapper: LayoutWrapper的draw方法
			- LayoutNodeWrapper依次向后调用，LayoutNodeWrapper绘制结束后，再依次调用LayoutNode的子节点的draw方法
		- 2、LayoutNode内部是分层绘制，每层最终由DrawModifier(Modifier子类）来进行绘制
		- 3、以上面的Demo为例，给Column底部添加颜色和文字
		  collapsed:: true
			- 1添加红色部分代码：
			  collapsed:: true
				- ![image.png](../assets/image_1684393906317_0.png)
			- 2、效果
			  collapsed:: true
				- ![image.png](../assets/image_1684393917094_0.png)
			- 3、跟踪drawBehind方法
			  collapsed:: true
				- ![image.png](../assets/image_1684393945789_0.png)
				- ![image.png](../assets/image_1684393953310_0.png)
			- 4、drawBehind方法最终创建了一个DrawBackgroundModifer(集成自DrawModifer)，并把DrawBackgroundModifer通过CombinedModifier连接在modifier链上
			  collapsed:: true
				- ![image.png](../assets/image_1684393973957_0.png)
			- 5、之后在给LayoutNode的modifer赋值的时候，判断如果modifier是DrawModifer就用ModifiedDrawNode包装到LayoutNodeWrapper链
			  collapsed:: true
				- ![image.png](../assets/image_1684393997623_0.png)
				-
			- 6、调用LayoutNode.draw方法时，从外向内执行LayoutNodeWrapper.performDraw方法，在performDraw方法执行绘制方法块
			  collapsed:: true
				- ![image.png](../assets/image_1684394014397_0.png)
		- 4、为什么关闭硬件加速后出现很多LayerView？
		  collapsed:: true
			- 1、我们看LayerView的实现的OwnedLayer。它定义了缩放、透明度、位移、阴影、旋转等多种ui属性
			  collapsed:: true
				- ![image.png](../assets/image_1684394096703_0.png)
			- 2、再看创建layer的代码，如果是硬件加速使用RenderNodeLayer，如果没有硬件加速会使用LayerView
			  collapsed:: true
				- ![image.png](../assets/image_1684394108762_0.png)
			- 3、调用Modifier.graphicsLayer { }和Modifier.graphicsLayer()会添加layer
				- 1、调用Modifier.graphicsLayer { } 在方法块里边修改ui属性
				- 2、Modifier.graphicsLayer() 在函数里边传递ui属性
- ## 布局流程
  collapsed:: true
	- compose的布局同样不使用android ui控件布局，是全新的一套布局流程。通过LayoutNode树完成布局，LayoutNode的大小和位置由LayoutNodeWrapper链来确定，最终都是通过Modifier来确定布局的
	- 1、从AndroidComposeView触发布局流程，MeasureAndLayoutDelegate来分发布局事件
	  collapsed:: true
		- ![image.png](../assets/image_1684394161112_0.png)
		- ![image.png](../assets/image_1684394167523_0.png)
		- ![image.png](../assets/image_1684394173988_0.png)
	- 2、测量流程
		- 1、测试最终也是从LayoutNodeWrapper层层测量来确定大小的，确定大小之后修改LayoutNodeWrapper相关的Layer大小
		  collapsed:: true
			- ![image.png](../assets/image_1684394200227_0.png)
		- 2、performMeasure最终调用LayoutModifier.measure来测量
		- 3、以Column为例
		  collapsed:: true
			- 3-1、通过modifier.requestHeight设置高度180
			  collapsed:: true
				- ![image.png](../assets/image_1684394225859_0.png)
			- 3-2、requestHeight实际会创建一个SizeModifier，并且链接到Modifier链
			  collapsed:: true
				- ![image.png](../assets/image_1684394247273_0.png)
			- 3-3、SizeModifier继承自LayoutModifer，并且实现了设置测量的功能
			  collapsed:: true
				- ![image.png](../assets/image_1684394266185_0.png)
			- 3-4、在LayoutNode遍历modifer的时候，遇到LayoutModifer后用ModifiedLayoutNode来包装LayoutModife
			  collapsed:: true
				- ![image.png](../assets/image_1684394280365_0.png)
			- 3-5ModifiedLayoutNode在测量时调用LayoutModifier去测量大小
			- 3-6LayoutModifer之后用自身的Constraints去继续向下层LayoutNodeWrapper测量
			- 3-7最终测量到最接近LayoutNode的InnerPlaceable（LayoutNodeWrapper子类），在这里会触发测试子LayoutNode
			  collapsed:: true
				- ![image.png](../assets/image_1684394307758_0.png)
			- 完成测量
		- 4、设置位置。
			- 1、测试完成之后会调用LayoutNode.place(或者replace)设置位置，最终执行到接口Placealble的placeAt方法
			- 2、先调用DelegatingLayoutNodeWrapper的placeAt方法，这里会调用LayoutNodeWrapper的placeAt方法，然后再调用measureResult.placeChildren去设置下层LayoutNodeWrapper的位置
				- ![image.png](../assets/image_1684394361583_0.png)
				- ![image.png](../assets/image_1684394367590_0.png)
	-
- ## 总结
	- compose是全新开发的ui框架，不依赖android ui
	- compose的ui树由LayoutNode构成
	- LayoutNode其实是简单的数据对象，实际的布局、绘制等都依赖Modifier链
	- 在缺少硬件加速的情况下，会存在很多ViewLayer的android view对象，他们的父类容器是ViewLayerContainer，ViewLayer都是兄弟节点
- ## 其他
	- 同城测试接入Jetpack Compose记录
	- [Compose优点](https://shimo.im/docs/qvvWyDRQYRK9kQrR/read)