- 方法一：借助buildSrc
- 方法二：启动远程调用
	- 1、Run-> Edit Configuration
	  collapsed:: true
		- ![image.png](../assets/image_1656397646365_0.png)
	- 2、新建远程调试器
	  collapsed:: true
		- ![image.png](../assets/image_1656398038260_0.png){:height 454, :width 655}
		- ![image.png](../assets/image_1656398182057_0.png){:height 850, :width 496}
	- 3、配置
	  collapsed:: true
		- ![image.png](../assets/image_1656398713483_0.png){:height 447, :width 655}
	- 4、修改task
	  collapsed:: true
		- ![image.png](../assets/image_1656398867425_0.png)
		- 将 suspend = n改为y
		- ![image.png](../assets/image_1656398948444_0.png)
	- 5、会生成如下配置，点击运行
		- ![image.png](../assets/image_1656399077747_0.png){:height 555, :width 655}
-