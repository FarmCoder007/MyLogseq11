title:: LayoutInflater.inflate 第三个参数作用

- LayoutInflater源码
	- ```java
	    public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
	      ....
	      final View temp = createViewFromTag(root, name, inflaterContext, attrs);
	  
	      ViewGroup.LayoutParams params = null;
	  
	      if (root != null) {
	        if (DEBUG) {
	          System.out.println("Creating params from root: " +
	                             root);
	        }
	        // 创建与根匹配的布局参数.如果xml里提供了
	        params = root.generateLayoutParams(attrs);
	        if (!attachToRoot) {
	          // Set the layout params for temp if we are not
	          // attaching. (If we are, we use addView, below)
	          temp.setLayoutParams(params);
	        }
	      }
	      }
	  ```
- root==null    构建的view将是一个独立的个体，属性无效
- root!=null  &&  attachToRoot[[#red]]==**false**==
                       1.属性值会依托于root构建，所以此时的xml根布局的属性有效
                       2.[[#red]]==**根布局产生的view不是root的子布局（不会添加到root上即第二个参数）**==
- root!=null  &&  attachToRoot==  **[[#red]]==true==**
                       1.属性值会依托于root构建，所以此时的xml根布局的属性有效
                       2.根布局产生的view是[[#red]]==**root的子布局，通过addView实现**==
-