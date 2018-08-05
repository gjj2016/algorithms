# 阿里测评 #
光明小学的小朋友们要举行一年一度的接力跑大赛了，但是小朋友们却遇到了一个难题：设计接力跑大赛的线路，你能帮助他们完成这项工作么？
光明小学可以抽象成一张有N个节点的图，每两点间都有一条道路相连。光明小学的每个班都有M个学生，所以你要为他们设计出一条恰好经过M条边的路径。

光明小学的小朋友们希望全盘考虑所有的因素，所以你需要把任意两点间经过M条边的最短路径的距离输出出来以供参考。

你需要设计这样一个函数：

res[][] Solve( N, M, map[][]);

注意：map必然是N * N的二维数组，且map[i][j] == map[j][i]，map[i][i] == 0，-1e8 <= map[i][j] <= 1e8。（道路全部是无向边，无自环）2 <= N <= 100, 2 <= M <= 1e6。要求时间复杂度控制在O(N^3*log(M))。

map数组表示了一张稠密图，其中任意两个不同节点i,j间都有一条边，边的长度为map[i][j]。N表示其中的节点数。

你要返回的数组也必然是一个N * N的二维数组，表示从i出发走到j，经过M条边的最短路径

你的路径中应考虑包含重复边的情况。
## 样例： ##
N = 3

M = 2

map = {

{0, 2, 3},

{2, 0, 1},

{3, 1, 0}

}
## 输出结果result为： ##
result = {

{4, 4, 3},

{4, 2, 5},

{3, 5, 2}

}
## 样例解释： ##
1->1有两种方法：1->2->1（长度为2+2=4），1->3->1（长度为3+3=6）

2->2有两种方法：2->1->2（长度为2+2=4），2->3->2（长度为1+1=2）

3->3有两种方法：3->1->3（长度为3+3=6），3->2->3（长度为1+1=2）

1->2只有一个方法：1->3->2（长度为3+1=4）

1->3只有一个方法：1->2->3（长度为2+1=3）

2->3只有一个方法：2->1->3（长度为2+3=5）

根据对称性可以得到其它部分的答案

## 代码 ##
	
	public class Main {
		public static void main(String[] args) {
			Scanner input = new Scanner(System.in);
			int n = input.nextInt();
			int m = input.nextInt();
			int n1 = input.nextInt();
			int n2 = input.nextInt();
			int[][] map = new int[n1][n2];
			for (int i = 0; i < n1; i++) {
				for (int j = 0; j < n2; j++) {
					map[i][j] = input.nextInt();
				}
			}
			int[][] result = solve(n, m, map);
			for (int i = 0; i < n; i++) {
				for (int j = 0; j < n; j++) {
					System.out.print(result[i][j]+"  ");
				}
				System.out.println();
			}
			if(input != null) {
				input.close();
			}
		}
	
		/**
		 *  
		 *  int[] queue = new int[queueLen];
			int[] lens = new int[queueLen];
			int[] step = new int[queueLen];
			
			求1号节点到1,2,3...n号节点的最短距离过程：
				queue: 1 2 3 1 3 1 2
				lens:  0 2 3 4 3 6 4
				step:  0 1 1 2 2 2 2
			可以得出1号节点到1,2,3...n号节点的最短距离分别为4, 4, 3
		 * @param n
		 * @param m
		 * @param map
		 * @return
		 */
		private static int[][] solve(int n, int m, int[][] map) {
			int[][] result = new int[n][n];
			for (int i = 0; i < map.length; i++) {
				int queueLen = (int)Math.pow(2, m+1);
				int[] queue = new int[queueLen];
				int[] lens = new int[queueLen];
				int[] step = new int[queueLen];
				int head = 0, tail=0;
				queue[tail] = i;
				lens[tail] = 0;
				step[tail] = 0;
				tail++;
				while(true){
					for (int k = 0; k < map[0].length; k++) {
						
						if(k == queue[head]) {
							continue;
						}
						queue[tail] = k;
						lens[tail] = lens[head]+map[queue[head]][k];
						step[tail] = step[head]+1;
						tail++;
						if(step[tail-1] > m) {
							
							break;
						}
					}
					head++;
					if(step[tail-1] > m) {
						break;
					}
				}
				int[] min = new int[n];
				for (int j = 0; j < min.length; j++) {
					min[j] = Integer.MAX_VALUE;
				}
				for (int l = 0; l < n; l++) {
					for (int j = 0; j < step.length; j++) {
						if (step[j] != m) {
							continue;
						}
						if(queue[j]==l && lens[j]<min[l]) {
							min[l] = lens[j];
						}
					}
				}
				result[i] = min;
				
			}
			return result;
		}
	
		
	
	}
