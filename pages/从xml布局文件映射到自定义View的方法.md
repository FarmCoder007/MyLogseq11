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
- 法二、
	- custom.xml
		- ```
		  ```