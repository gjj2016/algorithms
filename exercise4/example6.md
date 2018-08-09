# 招聘会小礼品 #
小摩召开了一场招聘会，招聘会现场一共有N个人，Mobike公司给大家准备了一些小礼品。但是我们并不知道每个人具体喜欢什么，
现在库房共有M种小礼品，每种小礼品有Ci件，共N件。而我们大致知道每个人选择某种小礼品的概率，
即能知道Pij(编号为i的人选择第j种小礼品的概率)。现在所有人按编号依次领小礼品（第1个人先领，第N个人最后领），
领小礼品时，参加者会按照预先统计的概率告诉准备者自己想要哪一种小礼品，
如果该种小礼品在他之前已经发放完了则他会领不到小礼品，请帮我们计算出能能领到小礼品的期望人数。

## 输入描述: ##
第一行包含两个整数N(1≤N≤300),M(1≤M≤100)，用单个空格隔开。表示公有N个应聘者，M种小礼品。

第二行为M个整数，依次为Ci，第i种小礼品的个数。

接下来的N行，每行M个实数，依次为Pij，第i个人选择第j种小礼品的概率。


## 输出描述: ##
一行输出期望人数。结果保留1位小数。

## 输入例子1: ##
>2 2
>
>1 1
>
>0.3 0.7
>
0.7 0.3

## 输出例子1: ##
>1.6

### 例子说明1: ###
(样例解释：共有4种选择(1,1),(1,2),(2,1),(2,2)，概率分别为0.21、0.09、0.49、0.21，(1,1),(2,2)这两种选择只有1个人能拿到小礼品，(1,2),(2,1)这两种选择有2个人能拿到小礼品，所以期望为1*(0.21+0.21) + 2*(0.09+0.49) = 1.58，保留一位小数为1.6。)

## 思路 ##
本题要求能领到礼物的人数，等于发出的礼物数，等于全部礼物数 - 剩下的礼物数。

考虑某个礼物，假设有3个，总共有3个人，每个人拿走这个礼物的概率分别为：0.3， 0.5， 0.2

定义数组dp记录这个礼物剩余0个、1个、2个、3个的概率：dp长度应该为c[j]+1，dp[i]表示这个礼物剩余i个的概率

那么模拟一下每个人取一个礼物的情况下dp的变化过程：

0  1  2  3

0  0  0  1   % 最开始，剩3个礼物，则dp[3]=1，其它为0

0  0  0.3 0.7  %第一个人可以选择拿或者不拿这个礼物，那么剩3个的概率为dp[3]*0.3， 剩2个的概率为dp[3] *0.3 + dp[2] *0.7

......
## 代码 ##
	public class Main {
		
		public static void main(String[] args) {
			Scanner input = new Scanner(System.in);
			int n = input.nextInt();//number of people
			int m = input.nextInt();//number of presents
			int[] c = new int[m];
			for (int i = 0; i < m; i++) {
				c[i] = input.nextInt();
			}
			double[][] p = new double[n][m];
			for (int i = 0; i < n; i++) {
				for (int j = 0; j < m; j++) {
					p[i][j] = input.nextDouble();
				}
			}
			double e = 0;
			for (int j = 0; j < m; j++) {
				if(c[j] == 0) {
					continue;
				}
				//这个礼物剩0个、1个...c[j]个的概率
				double[] presentLeft = new double[c[j]+1];
				//最开始剩c[j]个
				presentLeft[c[j]] = 1;
				for (int i = 0; i < n; i++) {
					//剩0个的概率为之前就剩0个的概率加上之前剩1个的概率乘以这个人拿走这个礼物的概率
					presentLeft[0] = presentLeft[0] + presentLeft[1]*p[i][j];
					//剩1个到剩c[j]-1个礼物的概率为之前剩k+1个乘以拿走加上之前剩k个乘以不拿走的概率
					for (int k = 1; k < c[j]; k++) {
						presentLeft[k] = presentLeft[k]*(1-p[i][j]) + presentLeft[k+1]*p[i][j];
					}
					//剩c[j]个的概率为之前就剩c[j]个的概率乘以不拿走的概率
					presentLeft[c[j]] = presentLeft[c[j]]*(1-p[i][j]);
				}
				//这个礼物剩多少个。
				for (int i = 0; i < c[j]+1; i++) {
					e += i*presentLeft[i];
				}
			}
			//把剩下多少个礼物全部加到e上，表示所有剩余的礼物数
			System.out.printf("%.1f", n-e);
			
			if(input != null) {
				input.close();
			}
		}
	
	}