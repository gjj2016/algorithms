# 序列最小化 #
有一个长度为N的序列。一开始，这个序列是1, 2, 3,... n - 1, n的一个排列。

对这个序列，可以进行如下的操作：

每次选择序列中k个连续的数字，然后用这k个数字中最小的数字替换这k个数字中的每个数字。

我们希望进行了若干次操作后，序列中的每个数字都相等。请你找出需要操作的最少次数。


## 输入描述: ##
第一行：两个数字n, k，含义如题，满足2 <= k <= n <= 105；

第二行：n个数字，是1, 2, 3,...n的一个排列。


## 输出描述: ##

一个整数，表示最少的次数。

>输入例子1:


2 2

2 1

>输出例子1:


1

>输入例子2:


4 3

1 2 3 4

>输出例子2:

2
	

	/*
	首先要找到包含最小的一个k组，这个k组就是第一步，比如说1,2,3，然后再发散一下，或左或右，
	即xx1,3xx，每组发散只包含了k-1个其他的数组内容，也就可以理解为去掉k个之后的数量除以k-1，即（n-k）/（k-1）。
	*/
	import java.util.Scanner;
	
	public class Main{
	    public static void main(String[] args){
	        Scanner input = new Scanner(System.in);
	        int n = input.nextInt();
	        int k = input.nextInt();
	        int[] nums = new int[n];
	        for (int i=0; i<n; i++){
	            nums[i] = input.nextInt();
	        }
	        System.out.print((int)(Math.ceil((n-k)*1.0/(k-1))+1));
	    }
	}