# 数字比较 #
牛牛很喜欢对数字进行比较，但是对于3 > 2这种非常睿智的比较不感兴趣。上了高中之后，学习了数字的幂，他十分喜欢这种数字表示方法，比如xy。
由此，他想出了一种十分奇妙的数字比较方法，给出两个数字x和y，请你比较xy和yx的大小，如果前者大于后者，输出">"，小于则输出"<"，等于则输出"="。

## 输入描述: ##

两个数字x和y。

满足1 <= x,y <= 109


## 输出描述: ##

一个字符，">"，"<"或者"="。

>输入例子1:


2 2

>输出例子1:


=

>输入例子2:


2 4

>输出例子2:


=
	
	```java
	import java.util.Scanner;

	public class Main{
    public static void main(String[] args){
        Scanner input = new Scanner(System.in);
        int x = input.nextInt();
        int y = input.nextInt();
        if (x == y){
            System.out.print("=");
        }  else {
            double checkpoint = y * Math.log(x) / Math.log(y);
            if (checkpoint > x){
                System.out.print(">");
            } else if (checkpoint == x){
                System.out.print("=");
            } else if (checkpoint < x){
                System.out.print("<");
            }
        }

        if (input != null ){
            input.close();
        }
    }
	}
	```