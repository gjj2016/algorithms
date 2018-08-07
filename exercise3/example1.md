# 判断符号 #

对任意a，b，-100000≤a≤b≤100000如何判断a×（a+1）×（a+2）×...×b结果的符号。
## 代码 ##

	public class Main {
	
		public static void main(String[] args) {
			Scanner input = new Scanner(System.in);
			int a = input.nextInt();
			int b = input.nextInt();
			int count = 0;
			int i = a;
			boolean flag = true;
			while (i <= 0 && i<=b) {
				if(i==0) {
					flag = false;
					System.out.println("Result is 0");
					break;
				}
				count++;
				i++;
			}
			if(flag) {
				if(count%2 == 0) {
					System.out.println("Positive");
				}else {
					System.out.println("Negtive");
				}
			}
			if (input != null) {
				input.close();
			}
		}
	
	}