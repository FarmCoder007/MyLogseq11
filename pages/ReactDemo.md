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
		  export default class Diff extends Component {
		  
		      constructor(props) {
		          super(props)
		  
		          this.state = {
		              inputOneVersion: "",
		              inputTwoVersion: "",
		              select: ""
		          }
		      }
		  
		     
		  
		      render() {
		          return (
		              <div>
		                  <header style={diffStyle.wraptocenter}>请先选择SDK</header>
		                  <select name="SDK选择器"
		                          onChange={event => this.selectOptionChange(event)}
		                          placeholder={"请选择要对比的SDK"}>
		                      {FeatureList.map((item, id) => (
		                          <option value={item.title}>{item.title}</option>
		                      ))}
		                  </select>
		                  <p>当前选择对比的SDK:{this.state.select}</p>
		              </div>
		          )
		      }
		  
		      /**
		       *  选择器变化
		       * @param event
		       */
		      selectOptionChange(event) {
		          this.setState({
		              select: event.target.value
		          });
		      }
		  }
		  
		  
		  ```