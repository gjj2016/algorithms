# 最小花费 #
有长度为length（0<length≤100000)的一个括号序列sequence，只有“（”或者“）”两种字符，每个括号的左右两边都能插一个括号，总共有length+1个位置可以插入括号，在第i个位置插入任意括号的代价是cost[i](0<cost[i]≤10000)同一个位置只能插入一个括号，求使得括号序列合法的最小代价。
## 输入示例1 ##
>length=6,  sequence="()))((", cost=[1,2,5,5,3,4,1]
## 输出示例1 ##
>8
## 输入示例2 ##
>length=6,  sequence="()))))", cost=[1,2,5,5,3,4,1]
## 输出示例2 ##
>10
>## 输入示例1 ##
>length=6,  sequence="()((((", cost=[1,2,5,5,3,4,1]
## 输出示例1 ##
>13
## 代码 ##
	public class CheapestSqe {
	
		public static void main(String[] args) {
			List<Integer> cost = new ArrayList<>();
			cost.add(1);
			cost.add(2);
			cost.add(5);
			cost.add(5);
			cost.add(3);
			cost.add(4);
			cost.add(1);
			System.out.println(getMinimumCost(6, "()((((", cost));
		}
	
		public static int getMinimumCost(int length, String sequence, List<Integer> cost) {
			String sqe = "";
			for (int i = 0; i < length; i++) {
				sqe += sequence.charAt(i);
			}
			sqe = clear(sqe);
			String tmp = sqe.replaceAll("00", "");
			int result = 0;
			// "((((....(((("
			//挑选从第一个'('开始的最小k个cost元素
			if (tmp.charAt(0) == '(') {
				int size = tmp.length();
				int ind = sqe.indexOf("(");
				int sum1 = top_k_min(cost, size, ind + 1);
				result += sum1;
			} else if (!tmp.contains("(")) {
				// "))))....))))"
				//挑选从最后一个')'往前的元素中最小的k个cost元素
				int size = tmp.length();
				int ind = sqe.lastIndexOf(")");
				int sum2 = top_k_min2(cost, size, ind);
				result += sum2;
			} else {
				// "...))))((((..."
				//找到')'和'('的分界下标，从第一个'('开始往后找最小k个cost元素，从最后一个')'往前找最小的k个cost元素
				int ind = sqe.indexOf("(");
				int count = 0;
				for (int i = ind; i < length; i++) {
					if (sqe.charAt(i) == '(') {
						count++;
					}
				}
				int sum3 = top_k_min(cost, count, ind + 1);
				int sum4 = top_k_min2(cost, tmp.length() - count, ind - 1);
				result += sum3 + sum4;
			}
			return result;
		}
	
		private static int top_k_min2(List<Integer> cost, int size, int i) {
			int result = 0;
			int[] array = new int[cost.size()];
			for (int j = 0; j < array.length; j++) {
				array[j] = cost.get(j);
			}
			Arrays.sort(array, 0, i + 1);
			//print(array);
			for (int j = 0; j < size; j++) {
				result += array[j];
			}
			return result;
		}
	
		private static int top_k_min(List<Integer> cost, int size, int i) {
			int result = 0;
			int[] array = new int[cost.size()];
			for (int j = 0; j < array.length; j++) {
				array[j] = cost.get(j);
			}
			Arrays.sort(array, i, array.length);
			//print(array);
			for (int j = 0; j < size; j++) {
				result += array[i + j];
			}
			return result;
		}
	
		// 去掉所有的合法括号，用0填充.
		private static String clear(String sqe) {
			while (sqe.contains("()")) {
				int index = sqe.indexOf("()");
				sqe = sqe.substring(0, index) + "00" + sqe.substring(index + 2);
			}
			return sqe;
		}
	
		public static void print(int[] array) {
			for (int i = 0; i < array.length; i++) {
				System.out.print(array[i] + "  ");
			}
			System.out.println();
		}
	}