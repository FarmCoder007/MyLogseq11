- # 一、input
	- 1-1：监听输入值变化
	  collapsed:: true
		- ```js
		  
		  export default class Diff extends Component {
		  
		      constructor(props) {
		          super(props)
		  
		          this.state = {
		              inputOneVersion: "",
		              inputTwoVersion: "",
		              select: ""
		          }
		      }
		  
		      /**
		       *
		       */
		      getValue(event) {
		          this.setState({
		              inputOneVersion: event.target.value
		          });
		      }
		  
		      render() {
		          return (
		              <div>
		                  <input onChange={this.getValue.bind(this)}/>
		                  <p>输入值变化:{this.state.inputOneVersion}</p>
		              </div>
		          )
		      }
		  }
		  
		  ```
- # 二、select
	- 1-1：监听选择器的变化
		- ```js
		  ```