- 方式一、xml
	- ```java
	  <androidx.constraintlayout.widget.ConstraintLayout
	      xmlns:android="http://schemas.android.com/apk/res/android"
	      xmlns:app="http://schemas.android.com/apk/res-auto"
	      xmlns:tools="http://schemas.android.com/tools"
	      android:layout_width="match_parent"
	      android:layout_height="match_parent"
	      tools:context=".MainActivity">
	  
	      <fragment
	          android:id="@+id/nav_host_fragment"
	          android:name="androidx.navigation.fragment.NavHostFragment"
	          android:layout_width="match_parent"
	          android:layout_height="match_parent"
	          app:defaultNavHost="true"
	          app:navGraph="@navigation/nav_graph" />
	  
	  </androidx.constraintlayout.widget.ConstraintLayout>
	  ```
- 方式二、
	- ```kotlin
	          val finalHost = NavHostFragment.create(R.navigation.nav_graph_main)
	          supportFragmentManager.beginTransaction()
	                  .replace(R.id.ll_fragment_navigation, finalHost)
	                  .setPrimaryNavigationFragment(finalHost)
	                  .commit();
	  ```
- 方式三
	- ```kotlin
	      @Override
	      public boolean onSupportNavigateUp() {
	          Fragment fragment = getSupportFragmentManager().findFragmentById(R.id.my_nav_host_fragment);
	          return NavHostFragment.findNavController(fragment).navigateUp();
	      }
	  ```