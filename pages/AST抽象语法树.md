- AST(Abstract syntax tree)抽象语法树解析
	- AST（Abstract syntax tree）的核心就是把Code转化为一种描述（对象、方法、运算符、流程控制语句、声明/赋值、内部类等），让我们可以在运行时比较方便的查看、修改Code，其中转化过程可以认为一种运行时编译。
- 作用方式
  collapsed:: true
	- ![image.png](../assets/image_1684397753467_0.png)
- 身边的应用
  collapsed:: true
	- IDE里面的Lombok插件、语法高亮、格式化代码、自动补全、代码混淆压缩都是利用了AST。
	- 线上体验AST：https://astexplorer.net/
- 利用AST扩展APT
	- 按照我们之前的了解，注解处理器APT只能用来生成代码，无法修改代码，但有了AST就不一样了。
	  IDE里面的Lombok插件的原理就是利用APT动态修改AST，给现有类增加了新的逻辑。
	- 原理我们现在都明白了，下面直接写个Hello World试试水。