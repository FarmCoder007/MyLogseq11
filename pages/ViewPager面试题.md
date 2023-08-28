## 1、viewPager和viewPager2的区别？
	- 底层实现不一样
		- 1、viewPager是自定义的viewgroup
		- 2、viewPager2底层使用Recyclerview
	- 懒加载实现不一样
		- 1、viewPager借助 setUserVisibleHint，和一些列标志位
		- 2、viewPager2使用lifeCycler
- ## 2、FragmentPagerAdapter和state的区别
- ## 3、[[Fragment的懒加载]]
-