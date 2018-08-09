## 子图个数 ##
【编程题】一个只包含0和1的阵列，找到1的组的个数，每个组的定义是横向和纵向相邻的值都为1

## 代码 ##
	public class NumberOfGroupOne {
		
		public static void main(String[] args) {
			getGroupNum(new int[][] {{1, 1, 0, 0, 1}, {1, 0, 0, 1, 0}, {1, 1, 0, 1, 0}, {0, 0, 1, 0, 0}});
		}
		
		public static int getGroupNum(int[][] map) {
			int n = map.length;
			int m = map[0].length;
			int[][] book = new int[n][m];
			int color = -1;
			for (int i = 0; i < n; i++) {
				for (int j = 0; j < m; j++) {
					 if(map[i][j] == 1) {
						 book[i][j] = 1;
						 dfs(i, j, map, book, color);
						 color--;
					 }
				}
			}
			System.out.println(-(++color));
			return 0;
		}
	
		private static void dfs(int i, int j, int[][] map, int[][] book, int color) {
			map[i][j] = color;
			int[][] direction = new int[][] {{1, 0}, {0, 1}, {-1, 0}, {0, -1}};
			for (int k = 0; k < 4; k++) {
				int next_i = i + direction[k][0];
				int next_j = j + direction[k][1];
				if(next_i<0 || next_i>map.length-1 || next_j<0 || next_j>map[0].length-1) {
					return;
				}
				if(book[next_i][next_j]==0 && map[next_i][next_j]==1) {
					book[next_i][next_j] = 1;
					dfs(next_i, next_j, map, book, color);
				}
			}
		}
	
	}
