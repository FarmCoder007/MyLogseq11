- 1、参考唯一性是根据Master分支的类大纲文件
	- 首次生成
		- 如果本地未获取到对应的类大纲集合文件，则认为是首次生成，然后会遍历当前工程的sourceSet目录，生成工程的类大纲集合文件。
		- 在MetaX框架中这个文件被命名为[[#red]]==api-public-method.json==，放在工程的根目录中，会随着Git提交到[[#red]]==master主分支==。
	- 需求合并时更新
		- 另外在RD每次合并当前需求分支时，会伴随的api变更检测Task[[#red]]==**自动重新生成当前类大纲集合文件**==，重新覆盖旧文件。这样就能解决数据源参考唯一性的问题。
	- [[#red]]==所以每次是需求分支的类大纲和master分支的类大纲做对比，得出api变更范围==
- 1、获取本次变更的类大纲文件保存到json里
	- Git仓库有如下三个主要部分组成：Remote（远程仓库）、Repository（本地仓库）、workspace（本地工作区）
	  id:: 64ec47e1-bfde-47bc-beee-76d401c75f75
	- Repository（本地仓库）提交还没合并到远端的
		- 因此使用git diff来显示本地分支与远程master分支不一样的地方。
	- workspace（本地工作区）未提交的
		- git status -s 会有修改，添加文件这样的标记，就可以获取修改列表
	- 2个数据源合并就是变更的
- 3、通过JSON数据对比，产出变更项