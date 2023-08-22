- [databinding.xmind](../assets/databinding_1691678836150_0.xmind)
- # 一、使用
	- ##  1、配置
		- 在使用的module和app的module都加入dataBinding{ enabled true}
		- ### 布局里需要加layout标签
		  collapsed:: true
			- ```xml
			  <?xml version="1.0" encoding="utf-8"?>
			  <layout xmlns:android="http://schemas.android.com/apk/res/android">
			  
			      <data>
			          <import type="android.view.View" />
			          <import type="android.text." />
			          <import type="androidx.databindinTextUtilsg.ObservableField" />
			          <variable
			              name="viewModel"
			              type="com.xiangxue.news.homefragment.newslist.views.titleview.TitleViewModel" />
			      </data>
			  
			      <LinearLayout
			          android:layout_width="match_parent"
			          android:layout_height="wrap_content"
			          android:animateLayoutChanges="true"
			          android:orientation="vertical">
			  
			          <TextView
			              android:layout_margin="16dp"
			              android:id="@+id/item_title"
			              android:layout_width="match_parent"
			              android:layout_height="wrap_content"
			              android:ellipsize="end"
			              android:visibility="@{TextUtils.isEmpty(viewModel.title)?View.GONE:View.VISIBLE}"
			              android:maxLines="2"
			              android:text="@{viewModel.title}"
			              android:gravity="center_vertical"
			              android:textColor="#303030"
			              android:textSize="16sp"
			              android:textStyle="bold"/>
			  
			          <View
			              android:layout_width="match_parent"
			              android:layout_height="1px"
			              android:background="#303030" />
			      </LinearLayout>
			  </layout>
			  ```
	- ## [DataBinding使用详解](https://www.jianshu.com/p/6dfd0b704f75)
	- ## 3、[[使用简单示例]]
- # [[viewBinding和dataBinding区别]]
- # 原理
	- ## [[Databinding原理解析]]
- # [[DataBinding-面试]]