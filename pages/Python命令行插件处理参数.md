- [Python命令行插件ArgumentParser](https://blog.csdn.net/weixin_44856211/article/details/124454218)
- 一、使用步骤
	- 创建解析对象
	  parser = argparse.ArgumentParser()
	- 添加命令行参数和选项，每一个add_argument方法对应一个参数或选项
	  parser.add_argument()
	- 调用parse_args()方法进行解析；解析成功之后即可使用
	  parser.parse_args()
- 二、示例
	- ```python
	  if __name__ == "__main__":
	  
	    parser = argparse.ArgumentParser()
	    parser.add_argument("projectName", type=str,
	                        help="新建项目的名称\n1.必须为XXXLib或者XXXSDK的命名规范. \n2.可以加[58|Wbu|Wuba]三种前缀，如58XXXLib")
	    parser.add_argument("targetProjectName", help="生成到哪个目录下，默认当前")
	    args = parser.parse_args()
	  
	    print(
	      "----------------------\n projectName: " + args.projectName + "----------------------\n")
	    project_name = args.projectName
	    project_name_result = re.match(r'(.*)(Lib|SDK)$', project_name)
	    target_project_name = args.targetProjectName
	  ```