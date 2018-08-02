# 两个字符串的最长公共子串 #
问题：有两个字符串str和str2，求出两个字符串中最长公共子串长度。

比如：str=acbcbcef，str2=abcbced，则str和str2的最长公共子串为bcbce，最长公共子串长度为5。

## 输入描述 ##
输入两个字符串
## 输出描述 ##
输出这两个字符串的最长子串长度及对应的子串
## 输入示例1 ##
>acbcbcef
>
>abcbced

## 输出示例1 ##
>5
>
>bcbce

## 输入示例2 ##
>a

>a

## 输出示例2 ##
>1

>a


	import java.util.Scanner;
	
	public class LongestCommonSquence {
		public static void main(String[] args) {
			Scanner input = new Scanner(System.in);
			String s1 = input.next();
			String s2 = input.next();
			int n = s1.length();
			int m = s2.length();
			int[] lens = new int[n*m];
			int maxLen = 0;
			String[] sqe = new String[n*m];
			for (int i = 0; i < sqe.length; i++) {
				sqe[i] = "";
			}
			int k=0;
			for (int i = 0; i < n; i++) {
				int ind1 = i;
				int ind2 = 0;
				while(ind1<n && ind2<m) {
					char c = s1.charAt(ind1);
					char c2 = s2.charAt(ind2);
					if (c == c2) {
						ind1++;
						ind2++;
						lens[k]++;
						sqe[k] += c;
					} else {
						ind2++;
						k++;
					}
					if (lens[k] > maxLen) {
						maxLen = lens[k];
					}
				}
				if(n-i < maxLen) {
					break;
				}
			}
			System.out.println("最长公共子序列长度："+maxLen);
			System.out.println("最长子序列有: ");
			for (int i = 0; i < lens.length; i++) {
				if (lens[i] == maxLen) {
					System.out.println(sqe[i]);
				}
			}
			
			if(input != null) {
				input.close();
			}
		}
	
	}