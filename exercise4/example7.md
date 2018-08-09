# 排序次数 #
小摩有一个N个数的数组，他想将数组从小到大 排好序，但是萌萌的小摩只会下面这个操作：
任取数组中的一个数然后将它放置在数组的最后一个位置。
问最少操作多少次可以使得数组从小到大有序？

## 输入描述: ##
首先输入一个正整数N，接下来的一行输入N个整数。(N <= 50, 每个数的绝对值小于等于1000)


## 输出描述: ##
输出一行操作数

## 输入例子1: ##
>4
>
19 7 8 25

## 输出例子1: ##
>2

### 例子说明1: ###
19放到最后，25放到最后，两步完成从小到大排序

## 思路 ##
统计没有按顺序出现的元素的个数，就是需要的最少操作数。

没有按顺序出现可以理解如下：

90 5 45 7 8 25 19 16

没有按顺序出现的有90,45,25,19.操作方式肯定是先拿出19放到最后（每次都拿最小的放到最后，试想如果拿了较大的放到最后，下次拿更小的元素的时候，这个先前移动的较大的元素还要进一步移动），这样4次就可以完成排序了。没有按序出现的元素的个数就是需要操作的最小数。但是考虑一种情况，当最大元出现在数组最后时：

90 5 45 7 8 25 19 16 90 

没有按序出现的仍然是90,45,25,19。但是现在必须在完成上述操作之后，再将最后的这个90拿出来放到数组最后，也就是如果最大元素出现在数组的末尾，需要多移动一次。但是要考虑一种情况，即数组本来就是有序的：

1 2 4 4 5

此时需要移动0个元素。总结一下就是：最小操作次数就是没有按序出现的元素的个数+（最大元素在最后一位&&没有按序出现的元素的个数不为0）？1:0

## 代码 ##
	public class SortCount {
		
		public static void main(String[] args) {
			Scanner input = new Scanner(System.in);
			int n = input.nextInt();
			int[] array = new int[n];
			for (int i = 0; i < n; i++) {
				array[i] = input.nextInt();
			}
			int min=Integer.MAX_VALUE, max=Integer.MIN_VALUE;
			int minIndex = 0;
			for (int i = 0; i < n; i++) {
				if(array[i] < min) {
					min = array[i];
					minIndex = i;
				}
				if(array[i] > max) {
					max = array[i];
				}
			}
			int count = minIndex;
			for (int i = minIndex+1; i < n-1; i++) {
				if(array[i] > array[i+1]) {
					count++;
				}
				
			}
			if(array[n-1] == max && count!=0) {
				count++;
			}
			
			System.out.println(count);
			if(input != null) {
				input.close();
			}
		}
	
	}