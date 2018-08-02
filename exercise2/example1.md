## 约瑟夫环 ##
  约瑟夫环问题的原来描述为，设有编号为1，2，……，n的n(n>0)个人围成一个圈，从第1个人开始报数，报到m时停止报数，报m的人出圈，再从他的下一个人起重新报数，报到m时停止报数，报m的出圈，……，如此下去，直到所有人全部出圈为止。当任意给定n和m后，设计算法求n个人出圈的次序。
## 输入描述 ##
输入连个整数n和m，n表示编号的最大值，m表示每一次报数的最大值。
## 输出描述 ##
输出出圈的编号顺序
## 输入示例1 ##
>8 4

## 输出示例1 ##
>4  8  5  2  1  3  7  6  

## 输入示例2 ##
>10 5

## 输出示例1 ##
>5  10  6  2  9  8  1  4  7  3  



	import java.util.ArrayList;
	import java.util.List;
	import java.util.Scanner;
	
	public class Johnseph {
		public static void main(String[] args) {
			Scanner input = new Scanner(System.in);
			List<Integer> result = new ArrayList<>(); 
			int n = input.nextInt();
			int m = input.nextInt();
			int[] mark = new int[n];
			for (int i = 0; i < mark.length; i++) {
				mark[i] = i+1;
			}
			int index = 0;
			int k = 1;
			while(exist0(mark)) {
				while(k!=m) {
					
					k++;
					index++;
					if(index == n) {
						index = 0;
					}
					if(mark[index] == 0) {
						k--;
					}
				}
				k=0;
				result.add(mark[index]);
				mark[index] = 0;
			}
			for (int i = 0; i < result.size(); i++) {
				System.out.print(result.get(i)+"  ");
			}
			
			if (input != null) {
				input.close();
			}
		}
	
		private static boolean exist0(int[] mark) {
			for (int i = 0; i < mark.length; i++) {
				if (mark[i] != 0) {
					return true;
				}
			}
			return false;
		}
	
	}