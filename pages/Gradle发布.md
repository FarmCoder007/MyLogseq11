- # 方式一：“maven-publish” 插件
	- 简介：使用Maven格式发布的功能，是由 “maven-publish” 插件提供的。
	- “publishing” 插件在project上创建了一个名为 “publishing”的 [[发布-PublishingExtension]]类型的扩展。这个扩展提供了两个容器，一个叫publications，一个叫repositories。“maven-publish”插件适用于MavenPublication publications 和 MavenArtifactRepository 仓库。
	- publishing{}：deploy当前项目到仓库
		- ```groovy
		  ```
-
-