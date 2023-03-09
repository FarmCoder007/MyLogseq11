title:: kotlin.String

- 1、字符串大小写转换
  collapsed:: true
	- ```
	      var abc = "AbCdEfG"
	      abc.toUpperCase() //java写法 在Studio中会有横杠提示
	      abc.toLowerCase() //java写法 在Studio中会有横杠提示
	   
	      //首字符转成大写第一种
	      var str =  abc.capitalize() //这种转换是字符串首字母变大写 但是1.5后推荐另一种写法
	      
	      
	      //首字符转成大写第二种（推荐）
	      var str =  abc.replaceFirstChar { if (it.isLowerCase()) it.titlecase(Locale.ROOT) else it.toString() }
	   
	   
	      //字符串全部转成大写kotlin
	       var str = abc.uppercase(Locale.getDefault())
	      
	      
	      //字符串全部转成小写kotlin
	      var str = abc.lowercase(Locale.getDefault())
	      
	      // 首字母转小写
	      var str =  abc.replaceFirstChar { if (it.isLowerCase()) it.lowercase(Locale.getDefault() else it.toString() }
	  ```
- 2、ifEmpty：  如果是空则使用默认的值，否则返回自身
	- ```
	  原
	  
	  ```
-