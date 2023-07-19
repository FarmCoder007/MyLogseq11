- 结合SDK工具dx.bat
- 1、配置环境变量，重启AS
  collapsed:: true
	- ![image.png](../assets/image_1689677209320_0.png)
- 2、AS中输入dx会有命令提示
- 3、在放有class目录下边 执行命令
	- ![image.png](../assets/image_1689677436769_0.png)
	- ```java
	  dx --dex --output=output.dex  input.jar(或者全包名，支持多个class)
	  ```
	- ![image.png](../assets/image_1689677551836_0.png)