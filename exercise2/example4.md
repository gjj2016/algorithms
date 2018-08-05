# 尽量少付钱 #
小明认为某些数字不吉利，付账时会尽可能少的多付一些钱，使得价格中不包含这些不吉利数字，并且不出现0.例如，不吉利数字为1，4，7，8，商品价格为1000，小明实际支付2222.
实现程序，输入商品原来的价格price，不吉利数字集合unlucky numbers，求小明付账时的价格lucky price.
## 代码 ##
	public class Main {
		public static void main(String[] args) {
			List<Integer> unlucky_number = new ArrayList<>();
			unlucky_number.add(0);
			unlucky_number.add(1);
			unlucky_number.add(4);
			unlucky_number.add(7);
			unlucky_number.add(8);
			System.out.println(getLuckyPrice(3000, unlucky_number));
		}
		
		public static int getLuckyPrice(int price, List<Integer> unlucky_numbers){
		     List<Integer> all_num = new ArrayList<>();
		     for (int i = 0; i < 10; i++) {
		    	 all_num.add(i);
		     }
		     all_num.removeAll(unlucky_numbers);
		     String pri = price+"";
		     String result = "";
		     for (int i = 0; i < pri.length(); i++) {
		    	int c = (int)(pri.charAt(i)-48);
				if(unlucky_numbers.contains((Object)c)) {
					c = c+1;
					while(!all_num.contains((Object)(c))){
						c++;
					}
					result += c;
				}else {
					result += c;
				}
			}
		     
		     return Integer.parseInt(result);
		}
	
	}