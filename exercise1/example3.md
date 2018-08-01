# 括号匹配 #
一般的括号匹配问题是这样的：

给出一个字符串，判断这个括号匹配是不是合法的括号匹配。

如"((" 和 "())"都不是合法的括号匹配，但是"()()()"，"(()())()"等就是合法的括号匹配。

这个问题解决起来非常简单，相信大家都知道怎么解决。

现在给出一个加强版的括号匹配问题： 给出n个由括号 '(' 和 ‘)’ 组成的字符串，请计算出这些字符串中有多少对字符串满足si + sj是合法的括号匹配。如果si + sj和sj + si都是合法的括号匹配(i ≠ j)，那么这两种搭配都需要计入答案；如果对于si，si + si是合法的括号匹配，那么也需要计入答案。

## 输入描述: ##

第一行是一个整数n，表示字符串的个数；

接下来n行是n个非空字符串，全部由'('和')'组成。

1 <= n <= 3 * 105，字符串的长度之和不超过3 * 105。


# 输出描述: #

一个整数，表示满足条件的字符串对的数量。

>输入例子1:


3

()

(

)

>输出例子1:


2

>输入例子2:


5

(()

)))))

()()()

(((

))

>输出例子2:


1

	````java
	import java.util.Scanner;
	
	public class Main{
	    public static void main(String[] args){
	        Scanner input = new Scanner(System.in);
	        int n = input.nextInt();
	        String[] ss = new String[n];
	        String tmp = input.nextLine();
	        for (int i=0; i<n; i++){
	            ss[i] = input.nextLine();
	        }
	        int[] num = new int[n];
	        int num1 = 0;
	        for (int i=0; i<n; i++){
	            String s = clear(ss[i]);
	            if (s.length() == 0){
	                num1++;
	            }else {
	                if (s.charAt(0) == '('){
	                    num[i] = s.length();
	                }else if(s.charAt(0)==')' && !s.contains("(")){
	                    num[i] = -s.length();
	                }
	            }
	            
	        }
	        int num2 = 0;
	        for (int i=0; i<n; i++){
	            if (num[i] == 0){
	                continue;
	            }
	            for (int j=i+1; j<n; j++){
	                
	                if (num[i] + num[j] == 0){
	                    num2++;
	                }
	            }
	        }
	        System.out.print(num1*num1 + num2);
	        if (input != null){
	            input.close();
	        }
	    }
	    
	    public static String clear(String s){
	        while(s.contains("()")){
	            s = s.substring(0, s.indexOf("()"))+s.substring(s.indexOf("()")+2);
	        }
	        return s;
	    }
	}
	````