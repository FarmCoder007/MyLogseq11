- ## 一、作用概念：
	- project.evaluationDependsOn 是指在使用 Gradle 构建系统时，可以用于==定义项目之间的依赖关系==。具体来说，它是一种用于定义项目之间相互依赖的机制，以确保在构建项目时正确地评估任务的顺序。
	- 例如，在一个多模块项目中，如果模块B依赖于模块A的输出结果，那么可以使用project(":B").evaluationDependsOn(":A:build") 将 B 模块的构建任务建立在 A 模块的构建任务之后，以确保在构建 B 模块之前 A 模块已经构建完成。
	- 这个机制可以用于管理 Gradle 项目中的任务依赖关系，确保正确地构建项目，尤其是在涉及多个模块的大型项目中。
-