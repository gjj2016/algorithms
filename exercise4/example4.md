# 字符串的个数 #
编程题】将给定的数转换为字符串，原则如下：1对应 a，2对应b，…..26对应z，例如12258可以转换为"abbeh", "aveh", "abyh", "lbeh" and "lyh"，个数为5，编写一个函数，给出可以转换的不同字符串的个数。
## 思路： ##
采用动态规划思想，从后往前，先求一个字符情况的解，然后根据一个字符的情况下的解求两个字符的解，再根据两个字符的解求三个字符的解，以此递推到第一个字符，返回第一个字符的解就是对应的字符串个数。
具体求每一个字符的解的时候，应该考虑两种情况，第一种情况是单独考虑这个字符，此时的字符串个数是*count(i+1)*，第二种情况是考虑这个字符及其后一个字符，如果这个组合在10和26之间，那么此时的字符串个数是*count(i+2)*，最终这个字符对应的解就是*count(i+1) + count(i+2)*.
## 代码 ##
		public static void count(String s) {
			int len = s.length();
			int[] counts = new int[len];
			for (int i = len-1; i >= 0; i--) {
				int count = 0;
				if(s.charAt(i)>='1' && s.charAt(i)<='9') {
					if (i == len-1) {
						count++;
					} else {
						count += counts[i+1];
					}
				}
				if(i<len-1) {
					int num = Integer.parseInt(s.substring(i, i+2));
					if(num>=10 && num<=26) {
						if(i == len-2) {
							count++;
						} else {
							count += counts[i+2];
						}
					}
				}
				counts[i] = count;
			}
			System.out.println(counts[0]);
		}