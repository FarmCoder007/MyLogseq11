- android:name：NavHost 的类名。
	- ```java
	         android:name="androidx.navigation.fragment.NavHostFragment"
	  ```
- app:navGraph：将 NavHostFragment 与导航图相关联。导航图指定了此 NavHostFragment 中用户可以导航到的所有目的地。xml 名字
	- ```xml
	     app:navGraph="@navigation/nav_graph_main"
	  ```
-
- app:[[#red]]==**defaultNavHost="true"：确保 NavHostFragment 拦截系统后退按钮**==。 请注意，只有一个NavHost 可以是默认值。 如果在同一布局中有多个 NavHost（例如双窗格布局），请确保仅指定一个默认 NavHost。