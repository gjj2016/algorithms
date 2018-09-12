今天上课，老师教了小易怎么计算加法和乘法，乘法的优先级大于加法，但是如果一个运算加了括号，那么它的优先级是最高的。例如：
	
	1+2*3=7
	1*(2+3)=5
	1*2*3=6
	(1+2)*3=9
现在小易希望你帮他计算给定3个数a，b，c，在它们中间添加"+"， "*"， "("， ")"符号，能够获得的最大值。

## 输入描述: ##
一行三个数a，b，c (1 <= a, b, c <= 10)


## 输出描述: ##
能够获得的最大值

## 输入例子1: ##
1 2 3

## 输出例子1: ##
9

## code ##


	import java.util.Scanner;
	
	public class Main {
		
		public static void main(String[] args) {
			Scanner in = new Scanner(System.in);
			int a = in.nextInt();
			int b = in.nextInt();
			int c = in.nextInt();
			int sum = 0;
			if(a==1 && b==1 && c==1) {
				System.out.println(3);
				return;
			}
			if(a==1) {
				b += 1;
				if(c == 1) {
					b += 1;
					System.out.println(b);
					return;
				} else {
					System.out.println(b*c);
					return;
				}
			}
			if(c==1) {
				b += 1;
				if(a == 1) {
					b += 1;
					System.out.println(b);
					return;
				} else {
					System.out.println(b*a);
					return;
				}
			}
			if(b==1) {
				if(a<c) {
					a += 1;
				}else {
					c += 1;
				}
				System.out.println(a * c);
				return;
			}
			System.out.println(a*b*c);
		}
	
	}
