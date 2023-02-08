- [Python命令行插件ArgumentParser](https://blog.csdn.net/weixin_44856211/article/details/124454218)
- 一、使用步骤
	- 创建解析对象
	  parser = argparse.ArgumentParser()
	- 添加命令行参数和选项，每一个add_argument方法对应一个参数或选项
	  parser.add_argument()
	- 调用parse_args()方法进行解析；解析成功之后即可使用
	  parser.parse_args()