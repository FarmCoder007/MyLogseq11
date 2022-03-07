- 法一、
	- layout.xml
collapsed:: true
		- ```
		  <?xml version="1.0" encoding="utf-8"?>
		  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
		      android:id="@+id/custom2"
		      android:layout_width="match_parent"
		      android:layout_height="wrap_content">
		  
		      <TextView
		          android:id="@+id/text2"
		          android:layout_width="wrap_content"
		          android:layout_height="wrap_content"
		          android:text="This is Text2" />
		  </LinearLayout>
		  
		  ```
		-
	- Custom.java
collapsed:: true
		- ```
		  import android.content.Context;
		  import android.graphics.Color;
		  import android.support.annotation.Nullable;
		  import android.util.AttributeSet;
		  import android.view.LayoutInflater;
		  import android.widget.LinearLayout;
		  import android.widget.TextView;
		  
		  public class Custom extends LinearLayout {
		      LinearLayout custom2;
		      TextView text2;
		  
		      public Custom2(Context context) {
		          super(context);
		          initView(context);
		      }
		  
		      public Custom2(Context context, @Nullable AttributeSet attrs) {
		          super(context, attrs);
		          initView(context);
		      }
		  
		      private void initView(Context context) {
		          custom2 = (LinearLayout) LayoutInflater.from(context).inflate(R.layout.layout, this);
		          text2 = custom2.findViewById(R.id.text2);
		          text2.setTextColor(Color.BLUE);
		      }
		  }
		  ```
- 法二、onFinishInflate 里初始化
	- custom.xml
collapsed:: true
		-
		- ```
		  <?xml version="1.0" encoding="utf-8"?>
		  <com.demo.Custom xmlns:android="http://schemas.android.com/apk/res/android"
		      android:id="@+id/custom1"
		      android:layout_width="match_parent"
		      android:layout_height="wrap_content"
		      android:orientation="vertical">
		  
		      <TextView
		          android:id="@+id/text1"
		          android:layout_width="wrap_content"
		          android:layout_height="wrap_content"
		          android:text="This is Text 1" />
		  
		  </com.demo.Custom1>
		  
		  ```
		-
		-
		-
		-
	- Custom.java
		- ```
		  import android.content.Context;
		  import android.graphics.Color;
		  import android.util.AttributeSet;
		  import android.widget.LinearLayout;
		  import android.widget.TextView;
		  
		  public class Custom1 extends LinearLayout {
		      Custom1 custom1;
		      TextView textView;
		  
		  
		      public Custom1(Context context) {
		          super(context);
		      }
		  
		      public Custom1(Context context, AttributeSet attrs) {
		          super(context, attrs);
		      }
		  
		      @Override
		      protected void onFinishInflate() {
		          super.onFinishInflate();
		          initViews();
		      }
		  
		      private void initViews() {
		          textView = findViewById(R.id.text1);
		          textView.setTextColor(Color.RED);
		      }
		  }
		  
		  ```