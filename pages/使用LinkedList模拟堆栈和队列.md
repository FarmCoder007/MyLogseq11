## 分析
	- [[#red]]==栈：特点先进后出==  【子弹梭子】，
	- [[#red]]==队列：先进先出==【管子】
- ## 模拟堆栈（Stack）
  collapsed:: true
	- ### 先进后出、子弹梭子从头进从头取，涉及方法addFirst()  removeFirst
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
		  
		      public Object peek() {
		          return stack.getFirst();
		      }
		  
		      public boolean isEmpty() {
		          return stack.isEmpty();
		      }
		  
		      public int size() {
		          return stack.size();
		      }
		  
		      public void clear() {
		          stack.clear();
		      }
		  
		      public static void main(String[] args) {
		          StackExample stack = new StackExample();
		          stack.push("Element 1");
		          stack.push("Element 2");
		          stack.push("Element 3");
		  
		          System.out.println("Size: " + stack.size());
		          System.out.println("Top element: " + stack.peek());
		  
		          while (!stack.isEmpty()) {
		              System.out.println("Popped: " + stack.pop());
		          }
		      }
		  }
		  
		  ```
- ## 模拟队列（Queue）
  collapsed:: true
	- ### 先进先出，管子，从后进前出，涉及方法addLast，removeFirst
	  collapsed:: true
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
		  
		      public Object peek() {
		          return queue.getFirst();
		      }
		  
		      public boolean isEmpty() {
		          return queue.isEmpty();
		      }
		  
		      public int size() {
		          return queue.size();
		      }
		  
		      public void clear() {
		          queue.clear();
		      }
		  
		      public static void main(String[] args) {
		          QueueExample queue = new QueueExample();
		          queue.enqueue("Element 1");
		          queue.enqueue("Element 2");
		          queue.enqueue("Element 3");
		  
		          System.out.println("Size: " + queue.size());
		          System.out.println("Front element: " + queue.peek());
		  
		          while (!queue.isEmpty()) {
		              System.out.println("Dequeued: " + queue.dequeue());
		          }
		      }
		  }
		  
		  ```