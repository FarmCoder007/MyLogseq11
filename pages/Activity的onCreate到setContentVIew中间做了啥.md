## 初始化phoneWindow：注意不是创建，创建在attach就完成了
	- ** 系统会为Activity创建一个Window对象，它代表了Activity的窗口。在`onCreate`方法中，会通过调用`getWindow()`方法获取Window对象，进而进行一些窗口相关的配置，比如设置窗口的样式、主题等。
- ## **解析Intent：**
	- 从`getIntent()`方法获取传递给Activity的Intent，这个Intent包含了Activity启动时的参数和数据。
- ## 加载Activity主题：
	- ** 根据Manifest文件中的主题信息，加载对应的Activity主题。
- ## 创建布局：
	- 在onCreate方法中，一般会通过调用setContentView方法来设置Activity的布局。在调用setContentView之前，系统会根据布局文件的ID加载对应的XML布局资源，并构建View层次结构。
-