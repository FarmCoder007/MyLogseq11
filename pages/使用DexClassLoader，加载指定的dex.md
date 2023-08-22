# 1、[[首先将class文件打包成dex]]
- # 2、使用DexClassLoader加载这个dex
	- ![image.png](../assets/image_1692671802654_0.png)
	- 因为其他apk或者其他dex里的class，没办法直接通过反射拿到的。
	- 先通过类加载器加载出来。再通过反射调用