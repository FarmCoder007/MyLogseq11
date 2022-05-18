- 一、动态添加ColorFilter,替换颜色
	- ```
	          mRefreshLottieView?.addLottieOnCompositionLoadedListener {
	              mRefreshLottieView?.addValueCallback(KeyPath("**"),  //修改对应keypath的填充色的属性值
	                  LottieProperty.COLOR,
	                  SimpleLottieValueCallback<Int?> { color })
	          }
	  ```