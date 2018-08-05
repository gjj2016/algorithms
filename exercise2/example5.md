# 和为0的最长序列长度 #
一个一维维数组中只有1和-1，实现程序，求和为0的最长子串长度，并在注释中给出时间和空间复杂度。
## 思路 ##
运动动态规划解题，dp[i][j]表示从array[i]一直加到array[j]的和，那么有如下递推式：

	dp[i][j] = dp[i+1][j-1] + array[i] + array[j]   %从i一直加到j可以计算为：i位置的值加上从i+1到j-1的和再加上j位置的值

	if dp[i][j] == 0:

  		 max = Math.max(max, j-i+1)                  %从i加到j的和出现0， 此时包含j-i+1个元素
## 代码 ##
	public class Main {
		
		public static void main(String[] args) {
		}
		
		static int getLongestLength(List<Integer> array){
			int size = array.size();
			int max = 0;
			int[][] dp = new int[size][size];
			//相邻元素的和，作为动态规划的边界条件
			for (int i = 1; i < size; i++) {
				dp[i-1][i] = array.get(i-1) + array.get(i);
				if(dp[i-1][i] == 0) {
					max = 2;
				}
			}
			//每次取两个元素加上去,由于相邻的元素已经加在一起了，从len=3开始表示从i后的两个元素开始
			//当然也可以双重循环i:0~size-1, j:i~size-1
			//下面这种每次取两个元素，可以少做几次循环
			for(int len=3; len<size; len = len+2) {
				for(int i=0, j; (j=i+len)<size; i++) {
					dp[i][j] = array.get(i) + dp[i+1][j-1] + array.get(j);
					if(dp[i][j] == 0) {
						max = Math.max(max, j-i+1);
					}
				}
			}
			
			return max;
		}
	
	}
