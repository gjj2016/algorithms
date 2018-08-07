## 反转一个循环链表 ##
翻转一个环形的链表
## 代码 ##
		public class ReverseLoopLink {
			
			public static void main(String[] args) {
				
			}
			
			public static Node reverse(Node root) {
				if(root == null) {
					return null;
				}
				List<Integer> data = new ArrayList<>();
				Node p = root.getNext();
				while(p != root) {
					data.add(p.getValue());
					p = p.getNext();
				}
				
				for (int i = data.size()-1; i >= 0; i++) {
					Node node = new Node();
					node.setValue(data.get(i));
					p.setNext(node);
					p = node;
				}
				return root;
			}
		
		}
		
		class Node{
			private int value;
			private Node next;
			public int getValue() {
				return value;
			}
			public void setValue(int value) {
				this.value = value;
			}
			public Node getNext() {
				return next;
			}
			public void setNext(Node next) {
				this.next = next;
			}
		}