## 最大均值差 ##
0-100的N个数(数的值范围为0~100 1 < N <= 1000),分成两组A、B:怎样分|meanA-meanB|最大? 
# 代码 #
	public class Main {
		
		public static void main(String[] args) {
			getMaxMean(new int[] {1, 2, 3, 4});
		}
		
		public static void getMaxMean(int[] a) {
			int N = a.length;
			int sum = 0;
			for (int i = 0; i < N; i++) {
				sum += a[i];
			}
			double mean = sum*1.0 / N;
			//由于N可能很大，而数据都在0~100之间，使用桶排序
			int[] f = new int[101];
			for (int i = 0; i < N; i++) {
				f[a[i]]++;
			}
			int[] sortedA = new int[N];
			int k=0;
			for (int i = 0; i < f.length;i++) {
				while(f[i]>0) {
					sortedA[k] = i;
					k++;
					f[i]--;
				}
			}
			//从前往后划分A和B
			int sum1=0;
			for (int i = 0; i < N-1; i++) {
				sum1 += sortedA[i];
				double meanA = sum1*1.0 / (i+1);
				double meanB = (sum-sum1)*1.0 / (N-i-1);
				double abs = Math.abs(meanA - meanB);
				if(abs > mean) {
					mean = abs;
				}
			}
			System.out.println("Max abs(meanA - meanB) is: "+mean);
		}
	
	}