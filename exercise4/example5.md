#  字符串问题 #
小摩手里有一个字符串A，小拜的手里有一个字符串B，B的长度大于等于A，所以小摩想把A串变得和B串一样长，这样小拜就愿意和小摩一起玩了。
而且A的长度增加到和B串一样长的时候，对应的每一位相等的越多，小拜就越喜欢。比如"abc"和"abd"对应相等的位数为2，为前两位。
小摩可以在A的开头或者结尾添加任意字符，使得长度和B一样。现在问小摩对A串添加完字符之后，不相等的位数最少有多少位？

## 输入描述: ##
第一行 为字符串A，第二行 为字符串B，

A的长度小于等于B的长度，B的长度小于等于100。

字符均为小写字母。


## 输出描述: ##
输出一行整数表示A串添加完字符之后，A B 不相等的位数最少有多少位？

## 输入例子1: ##
>abe
>
cabc

## 输出例子1: ##
>1


## 思路： ##
类似于寻找两个字符串的最长相同子串，但是又不完全相同。寻找两个字符串可以使用如下方法：

   c  a  b  c

a  0  1  0  0

b  0  0  1  0

e  0  0  0  0

用dp[i][j]记录字符串A的某一位是否与字符串B的某一位相同，那么最长公共子串就是右下45°方向上连续1的个数。

但是考虑下面的情况：

A： axxbxxx

B： cxxxddd

此时就不能找最长相同子串了，造成的原因是字符串A必须往左移才能让相同子串对齐，这将会造成字符串A和B长度不一致。因此，求dp的时候对每一行不能从0开始，应该从**i**开始，在**i**之前和当前字符相同的不能考虑。

## 代码 ##
	public class Main {
		
		public static void main(String[] args) {
			Scanner input = new Scanner(System.in);
			String A = input.next();
			String B = input.next();
			int lena = A.length();
			int lenb = B.length();
			int[][] dp = new int[lena][lenb];
			for (int i = 0; i < lena; i++) {
				for (int j = i; j < lenb; j++) {
					if (A.charAt(i) == B.charAt(j)) {
						dp[i][j] = 1;
					}
				}
			}
			String maxIndex = "0,0";
			int maxLen = 0;
			for (int i = 0; i < lena; i++) {
				for (int j = 0; j < lenb; j++) {
					int count = 0;
					String index = i+","+j;
					if(dp[i][j] == 1) {
						int next_i = i+1;
						int next_j = j+1;
						count++;
						while(next_i<lena && next_j<lenb && dp[next_i][next_j]==1) {
							count++;
							next_i++;
							next_j++;
						}
					}
					if(count > maxLen) {
						maxLen = count;
						maxIndex = index;
					}
				}
			}
			int leftIndex = Integer.parseInt(maxIndex.split(",")[0]);
			int rightIndex = leftIndex + maxLen;
			int count = 0;
			for (int i = leftIndex-1; i >= 0; i--) {
				if(A.charAt(i) !=B.charAt(i)) {
					count++;
				}
			}
			for (int i = rightIndex; i < lena; i++) {
				if(A.charAt(i) != B.charAt(i)) {
					count++;
				}
			}
			System.out.println(count);
			if(input != null) {
				input.close();
			}
		}
	
	}