## 分析
	- [[#red]]==栈：特点先进后出==  【子弹梭子】，
	- [[#red]]==队列：先进先出==【管子】
- ## 模拟堆栈（Stack）
	- ### 先进后出、子弹梭子从头进从头取，涉及方法[[#red]]==**addFirst()  removeFirst**==
		- addFirst会一直在第一个位置上添加元素，之前添加的后移
		- removeFirst 移除的是后来添加的
		- ```java
		  import java.util.LinkedList;
		  
		  public class StackExample {
		      // 模拟堆栈
		      private LinkedList<Object> stack;
		  
		      public StackExample() {
		          stack = new LinkedList<>();
		      }
		  
		      // 添加元素 每次都添加到第一个
		      public void push(Object item) {
		          stack.addFirst(item);
		      }
		  
		       // 取出元素  每次都从第一个取出
		      public Object pop() {
		          return stack.removeFirst();
		      }
		  }
		  
		  ```
- ## 模拟队列（Queue）
	- ### 先进先出，管子，从后进前出，涉及方法[[#red]]==**addLast，removeFirst**==
		- ```java
		  import java.util.LinkedList;
		  
		  public class QueueExample {
		      private LinkedList<Object> queue;
		  
		      public QueueExample() {
		          queue = new LinkedList<>();
		      }
		      // 后进
		      public void enqueue(Object item) {
		          queue.addLast(item);
		      }
		      
		      // 前出
		      public Object dequeue() {
		          return queue.removeFirst();
		      }
		  }
		  
		  ```