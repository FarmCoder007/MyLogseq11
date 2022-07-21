- [官网](https://react.docschina.org/)
- # 一、简介
	- React 是用于创建页面的 UI 库
	- 官方推荐使用 一种JSX语法,类似xml
- # 二、api
- [React进阶React全部api解读](https://blog.csdn.net/weixin_43484007/article/details/124391509)
-
- # 三、JSX语法
  collapsed:: true
	- ## 简介
		- ```js
		  const element = <h1>Hello, world!</h1>;
		  ```
		- 这个有趣的标签语法既不是字符串也不是 HTML。
		- 它被称为 JSX，是一个 JavaScript 的语法扩展。我们建议在 React 中配合使用 JSX，JSX 可以很好地描述 UI 应该呈现出它应有交互的本质形式。JSX 可能会使人联想到模版语言，但它具有 JavaScript 的全部功能。
	-
- # 四、[[React常用api]]
- # 五、组件 & Props
	- 组件允许你将 UI 拆分为独立可复用的代码片段，并对每个片段进行独立构思。本指南旨在介绍组件的相关理念
- # 六、事件
	- ## onChange事件
		- ### 6-1、将 onChange 处理程序添加到输入
		  collapsed:: true
			- ```js
			  import React from 'react';
			  
			  function App() {
			  
			    // 
			    function handleChange(event) {
			      console.log(event.target.value);
			    }
			    
			    // 将onChange事件 传递到handleChange 函数中
			    return (
			      <input name="firstName" onChange={handleChange} />
			    );
			  }
			  
			  export default App;
			  ```
			- onChange 事件处理程序返回一个Synthetic Event对象，其中包含有用的元数据，例如目标输入的 id、名称和当前值。
			- 我们可以通过访问e.target.value来访问 handleChange 内部的目标输入值。因此，要记录输入字段的名称，我们可以记录e.target.name。
			- 上面的例子是一个功能组件。如果您使用的是 Class 组件，则必须将 onChange 事件处理程序绑定到this 的上下文。要了解有关功能组件和基于类的组件之间差异的更多信息，[请查看本指南。](https://upmostly.com/tutorials/react-functional-vs-class-components)
		- ### 6-2、在 React 组件中使用 onChange 将输入值保存在状态内部
			- 我们今天将探讨的最后一个示例是如何将输入的当前值存储在状态值中。我们将使用 React 提供给我们的 [useState](https://upmostly.com/tutorials/simplifying-react-state-and-the-usestate-hook) 钩子。您可以在此处了解有关 useState 挂钩的更多信息。
			- 这对于跟踪当前输入字段值并在 UI 上显示它们非常有用。
			- ```js
			  import React from 'react';
			  
			  function App() {
			    const [firstName, setFirstName] = useState('');
			    
			    return (
			      <input value={firstName}   name="firstName" onChange={e => setFirstName(e.target.value)} />
			    );
			  }
			  
			  export default App;
			  ```
			-