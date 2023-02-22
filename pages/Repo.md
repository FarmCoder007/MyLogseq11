- [官方文档](https://gerrit.googlesource.com/git-repo/)
- [官网](https://code.google.com/archive/p/git-repo/)
- # 一、简介
	- "Repo" 是一个用于管理多个 Git 仓库的工具。它是由 Google 开发的，旨在为管理 Android 平台的多个 Git 仓库提供一个更加简单和统一的工具。
	- Repo 提供了一个名为 "manifest" 的 XML 文件，其中包含了所有需要管理的 Git 仓库的信息，包括仓库的 URL、分支、提交 ID 等。使用 Repo 工具，可以根据 manifest 文件中的配置信息自动下载、更新和同步多个 Git 仓库。
	- 通过使用 Repo，可以方便地同时管理多个 Git 仓库，特别是在需要进行多个仓库之间的交叉开发或协同开发时非常有用。
- # 二、Repo 私域搭建
	- ## Repo的组成部分：
		- 1、Repo launcher (Repo 引导脚本)
		- 2、Repo 命令的主体部分
	- 这样设计旨在使 Repo launcher只包含最基本的init、help两个命令，主体命令可实现自动更新，这将显著改善用户的使用体验。
	- ##
	- 私域镜像过程：
	  
	  通过科学上网，从git-repo官网上，克隆最新代码到本地
	- # git 设置代理
	  git config --global http.proxy 'socks5://127.0.0.1:7890'
	  git config --global https.proxy 'socks5://127.0.0.1:7890'
	- # clone 仓库
	  git clone https://gerrit.googlesource.com/git-repo 
	  删除 .git 和 .github 文件夹（原因是igit限制了提交的邮箱后缀）
	  私域上创建git-repo项目，再提交到私域
	  修改 Repo 的部分代码
	  REPO_URL = os.environ.get('REPO_URL', None)
	  if not REPO_URL:
	   REPO_URL = 'https://gerrit.googlesource.com/git-repo'
	  REPO_REV = os.environ.get('REPO_REV')
	  if not REPO_REV:
	   REPO_REV = 'stable'
	  
	  // 改为如下：
	  REPO_URL = 'git@igit.58corp.com:git_mirror/git-repo.git'
	  REPO_REV = 'master'
	  Repo 私域安装步骤
	  下载 Repo launcher（即repo文件）：https://igit.58corp.com/git_mirror/git-repo/-/raw/master/repo?inline=false
	  修改 repo 的权限
	  chmod 777 xxx/repo
	  repo 配置环境变量
	- # ~/.zshrc
	  export PATH="xxx:$PATH"
	  Repo开发流
	  Repo 开发流：
	  
	  初始化：repo init
	  repo init -u git@igit.58corp.com:com.wuba.wuxian.android/manifest_58app.git -b release-xxx -g main,xxx,xxx
	  
	  -b 指定版本分支
	  -g 指定业务线，注意：main是必须的
	  同步代码：repo sync
	  创建本地分支：repo start f-xxx --all
	  Android Studio 开发
	  使用之前应该配置相应的Manifest文件，如下所示：
	  
	  <?xml version="1.0"?>
	  
	  <manifest>
	    <remote
	        name="58igit"
	        fetch="ssh://git@igit.58corp.com/"/>
	  
	    <default
	        revision="release-12.7.0"
	        remote="58igit"
	        sync-j="4"/>
	  
	    <!-- 存放CodeTure的项目 -->
	    <project
	        clone-depth="1"
	        name="com.wuba.wuxian.android/codeture_58app"
	        path="codeture_58app"
	        groups="main"/>
	  
	    <!-- 58App的文档项目 -->
	    <project
	        clone-depth="1"
	        name="wuxian-doc/58app"
	        path="doc_58app"
	        revision="master"
	        groups="main"/>
	  
	    <!-- docsify 文档平台项目 -->
	    <project
	        clone-depth="1"
	        name="wuxian-doc/doc_docsify"
	        path="doc_docsify"
	        revision="master"
	        groups="main"/>
	  
	    <!-- 58App的入口工程 -->
	    <project
	        clone-depth="1"
	        name="com.wuba.wuxian.android/58ClientProject"
	        path="58ClientProject"
	        groups="main"/>
	  
	    <project
	        clone-depth="1"
	        name="com.wuba.wuxian.android.main/58WuxianClient"
	        path="58ClientProject/58WuxianClient"
	        groups="main"/>
	  
	    <project
	        clone-depth="1"
	        name="com.wuba.wuxian.android.main/58TownClient"
	        path="58ClientProject/58TownClient"
	        groups="main"/>
	  
	    <project
	        clone-depth="1"
	        name="com.wuba.wuxian.android.main/58ClientHybridLib"
	        path="58ClientProject/58ClientHybridLib"
	        groups="core"/>
	  
	    <project
	        clone-depth="1"
	        name="com.wuba.wuxian.android.house/58HouseLib"
	        path="58ClientProject/58HouseLib"
	        groups="house"/>
	  
	    <!-- Hybrid sdk -->
	    <project
	        clone-depth="1"
	        name="com.wuba.wuxian.sdk/WubaHybridSDK"
	        path="58ClientProject/WubaHybridSDK"
	        groups="sdk"
	        revision="refs/tags/1.6.12"/>
	  
	  </manifest>
	  
	  使用场景
	  场景1：管理58App相关的所有Git仓库
	  58App相关的所有Git仓库，不仅仅代码仓库，还有很多的工具仓库，如doc文档、CodeTure、Python比对脚本等等
	  
	  统一配置到manifest之后，可带来如下好处：
	  
	  快速找到所有相关的仓库
	  实现全局搜索，按 Google 内部的调研，发现平均每位工程师每天会执行 12 个代码搜索请求
	  代码阅读笔记可实现快速共享
	  创建一个manifest仓库，依据58App的发版方式，创建对应的版本分支，每个分支下，配置相应的default.xml文件
	  
	  场景2：管理各种厂商预装包的特殊要求
	  类似Flipper一样，不用去分散的写各种python脚本，解决工具查找的问题。
	  
	  manifest的配置如上一样
	  
	  场景3：管理各种有业务差异的马甲包，如早期的58同城与58本地
	  一体化项目之前，58本地fork了一些底层库，导致在下载相应源码库时，容易出现混淆。