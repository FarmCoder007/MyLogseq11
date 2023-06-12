title:: 剑指 Offer 40. 最小的k个数-简单

- # 题目
	- 输入整数数组 `arr` ，找出其中最小的 `k` 个数。例如，输入4、5、1、6、2、7、3、8这8个数字，则最小的4个数字是1、2、3、4。
	- ## 示例1
		- ```java
		  输入：arr = [3,2,1], k = 2
		  输出：[1,2] 或者 [2,1]
		  ```
	- ## 示例
		- ```java
		  输入：arr = [0,1,2,1], k = 1
		  输出：[0]
		  ```
	- **限制：**
		- `0 <= k <= arr.length <= 10000`
		- `0 <= arr[i] <= 10000`
- # 思路
	- 先排序（本题基于快排）
	- 后取前几个数
- ## 代码
	- ```java
	  class Solution {
	      public int[] getLeastNumbers(int[] arr, int k) {
	          quickSort(arr, 0, arr.length - 1);
	          return Arrays.copyOf(arr, k);
	      }
	    
	      // 快速排序
	      private void quickSort(int[] arr, int left, int right) {
	          // 子数组长度为 1 时终止递归
	          if (left >= right) return;
	          // 哨兵划分操作（以 arr[left] 作为基准数）
	          int i = left, j = right;
	          while (i < j) {
	              // 从右向左 查找 首个小于基准数的元素
	              while (i < j && arr[j] >= arr[left]) j--;
	              // 从左向右 查找 首个大于基准数的元素
	              while (i < j && arr[i] <= arr[left]) i++;
	              // 交换 aar[i]  arr[j]
	              swap(arr, i, j);
	          }
	          // 交换基准点到中间
	          swap(arr, i, left);
	          // 递归左（右）子数组执行哨兵划分
	          quickSort(arr, left, i - 1);
	          quickSort(arr, i + 1, right);
	      }
	      // 交换
	      private void swap(int[] arr, int i, int j) {
	          int tmp = arr[i];
	          arr[i] = arr[j];
	          arr[j] = tmp;
	      }
	  }
	  ```
- # 复杂度
	- 时间：O(NlogN)
	- 空间：O(N)