# 字符迷阵 #
字符迷阵是一种经典的智力游戏。玩家需要在给定的矩形的字符迷阵中寻找特定的单词。
在这题的规则中，单词是如下规定的：

1. 在字符迷阵中选取一个字符作为单词的开头；

2. 选取右方、下方、或右下45度方向作为单词的延伸方向；
3. 以开头的字符，以选定的延伸方向，把连续得到的若干字符拼接在一起，则称为一个单词。


现在的问题是，给出一个字符迷阵，及一个要寻找的单词，问能在字符迷阵中找到多少个该单词的合法方案。注意合法方案是可以重叠的
## 输入描述: ##
输入的第一行为一个正整数T，表示测试数据组数。 接下来有T组数据。每组数据的第一行包括两个整数m和n，表示字符迷阵的行数和列数。接下来有m行，每一行为一个长度为n的字符串，按顺序表示每一行之中的字符。再接下来还有一行包括一个字符串，表示要寻找的单词。  数据范围： 对于所有数据，都满足1<=T<=9，且输入的所有位于字符迷阵和单词中的字符都为大写字母。要寻找的单词最短为2个字符，最长为9个字符。字符迷阵和行列数，最小为1，最多为99。 对于其中50%的数据文件，字符迷阵的行列数更限制为最多为20。


## 输出描述: ##
对于每一组数据，输出一行，包含一个整数，为在给定的字符迷阵中找到给定的单词的合法方案数。

## 输入例子1: ##
>3
>
10 10

>AAAAAADROW
>
>WORDBBBBBB
>
>OCCCWCCCCC
>
>RFFFFOFFFF
>
>DHHHHHRHHH
>
>ZWZVVVVDID
>
>ZOZVXXDKIR
>
>ZRZVXRXKIO
>
>ZDZVOXXKIW
>
>ZZZWXXXKIK
>
WORD

>3 3
>
>AAA
>
>AAA
>
>AAA
>
>AA
>
>5 8
>
>WORDSWOR
>
>ORDSWORD
>
>RDSWORDS
>
>DSWORDSW
>
>SWORDSWO
>
SWORD

## 输出例子1: ##
>4
>
>16
>
5

## 代码 ##
	import java.util.Scanner;
	public class Main {
		
		public static void main(String[] args) {
			Scanner input = new Scanner(System.in);
			int T = input.nextInt();
			while(T > 0) {
				int m = input.nextInt();
				int n = input.nextInt();
				String[] map = new String[m];
				for (int i = 0; i < m; i++) {
					map[i] = input.next();
				}
				String word = input.next();
				char start = word.charAt(0);
				int len = word.length();
				int count = 0;
				for (int i = 0; i < m; i++) {
					for (int j = 0; j < n; j++) {
						if(map[i].charAt(j) != start) {
							continue;
						} 
						//横着统计
						String tmp = "";
						for (int k = 0, l=j; l<n&&k < len; k++, l++) {
							tmp += map[i].charAt(l);
						}
						if(word.equals(tmp)) {
							count++;
						}
						//竖着统计
						tmp = "";
						for(int k=0, l=i; l<m&&k<len; k++, l++) {
							tmp += map[l].charAt(j);
						}
						if(word.equals(tmp)) {
							count++;
						}
						//右下方统计
						tmp = "";
						for(int k=0, l=i, p=j; l<m&&p<n&&k<len; l++, p++, k++) {
							tmp += map[l].charAt(p);
						}
						if(word.equals(tmp)) {
							count++;
						}
					}
				}
				System.out.println(count);
				T--;
			}
			
			if(input != null) {
				input.close();
			}
		}
	
	}