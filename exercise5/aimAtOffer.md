**1、题目描述**

在一个二维数组中（每个一维数组的长度相同），每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。
请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

	public class Solution {
	    public boolean Find(int target, int [][] array) {
	        if(array == null || array.length==0){
	            return false;
	        }
	        int i = array.length-1;
	        int j = 0;
	        for(;i>=0&&j<array[0].length;){
	            if(target == array[i][j]){
	                return true;
	            } else if(target > array[i][j]){
	                j++;
	            } else{
	                i--;
	            }
	        }
	        return false;
	    }
	}

**2、题目描述**

请实现一个函数，将一个字符串中的每个空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。

	public class Solution {
	    public String replaceSpace(StringBuffer str) {
	    	if(str == null){
	            return null;
	        }
	        if(str.length() == 0){
	            return "";
	        }
	        return str.toString().replaceAll(" ", "%20");
	    }
	}


**3、输入一个链表，按链表值从尾到头的顺序返回一个ArrayList。**
	
	/**
	*    public class ListNode {
	*        int val;
	*        ListNode next = null;
	*
	*        ListNode(int val) {
	*            this.val = val;
	*        }
	*    }
	*
	*/
	import java.util.ArrayList;
	public class Solution {
	    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
	        ArrayList<Integer> result = new ArrayList<>();
	        if(listNode == null){
	            return result;
	        }
	        ArrayList<Integer> list = new ArrayList<>();
	        ListNode head = listNode;
	        while(head != null){
	            list.add(head.val);
	            head = head.next;
	        }
	        
	        for(int i=list.size()-1; i>=0; i--){
	            result.add(list.get(i));
	        }
	        return result;
	    }
	}

**4、题目描述**

输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。

假设输入的前序遍历和中序遍历的结果中都不含重复的数字。

例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。
	
	/**
	 * Definition for binary tree
	 * public class TreeNode {
	 *     int val;
	 *     TreeNode left;
	 *     TreeNode right;
	 *     TreeNode(int x) { val = x; }
	 * }
	 */
	public class Solution {
	    public TreeNode reConstructBinaryTree(int [] pre,int [] in) {
	        TreeNode root = new TreeNode(pre[0]);
	        if(pre.length == 1 || in.length == 1){
	            return root;
	        }
	        int index = 0;
	        for(int i=0; i<in.length; i++){
	            if(in[i] == pre[0]){
	                index = i;
	            }
	        }
	        if(index > 0){
	            int[] prev = new int[index];
	            for(int i=1; i<prev.length+1; i++){
	                prev[i-1] = pre[i];
	            }
	            int[] mid = new int[index];
	            for(int i=0; i<mid.length; i++){
	                mid[i] = in[i];
	            }
	            root.left = reConstructBinaryTree(prev, mid);
	        }
	        if(index<in.length-1){
	            int[] prev = new int[in.length-1-index];
	            for(int i=0; i<prev.length; i++){
	                prev[i] = pre[i+index+1];
	            }
	            int[] mid = new int[in.length-1-index];
	            for(int i=index+1; i<in.length; i++){
	                mid[i-index-1] = in[i];
	            }
	            root.right = reConstructBinaryTree(prev, mid);
	        }
	        return root;
	    }
	}

**5、题目描述**

用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。

	import java.util.Stack;
	
	public class Solution {
	    Stack<Integer> stack1 = new Stack<Integer>();
	    Stack<Integer> stack2 = new Stack<Integer>();
	    
	    public void push(int node) {
	        stack1.push(node);
	    }
	    
	    public int pop() {
	        while(!stack1.isEmpty()){
	            int x = stack1.pop();
	            stack2.push(x);
	        }
	        int result = stack2.pop();
	        while(!stack2.isEmpty()){
	            stack1.push(stack2.pop());
	        }
	        return result;
	    }
	}


**6、题目描述**

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。 输入一个非减排序的数组的一个旋转，输出旋转数组的最小元素。
 
例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。 

NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。
	
	import java.util.ArrayList;
	public class Solution {
	    public int minNumberInRotateArray(int [] array) {
	        if(array == null || array.length==0){
	            return 0;
	        }
	        for(int i=array.length-1; i>=1; i--){
	            if(array[i] < array[i-1]){
	                return array[i];
	            }
	        }
	        return array[0];
	    }
	}

**7、题目描述**

大家都知道斐波那契数列，现在要求输入一个整数n，请你输出斐波那契数列的第n项（从0开始，第0项为0）。


	public class Solution {
	    public int Fibonacci(int n) {
	        if(n == 0){
	            return 0;
	        } else if(n==1){
	            return 1;
	        } else{
	            return Fibonacci(n-1) + Fibonacci(n-2);
	        }
	    }
	}
	运行时间：1336ms
	占用内存：9272k
	
	public class Solution {
	    public int Fibonacci(int n) {
	        if(n == 0){
	            return 0;
	        }
	        int[] f = new int[n+1];
	        f[0] = 0;
	        f[1] = 1;
	        for(int i=2; i<n+1; i++){
	            f[i] = f[i-1] + f[i-2];
	        }
	        return f[n];
	    }
	}
	非递归版时间大大减少
	运行时间：17ms
	占用内存：9272k


**8、题目描述**

一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法（先后次序不同算不同的结果）。

	public class Solution {
	    public int JumpFloor(int target) {
	        if(target == 1){
	            return 1;
	        }
	        if(target == 2){
	            return 2;
	        }
	        return JumpFloor(target-1) + JumpFloor(target-2);
	    }
	}

**9、题目描述**

一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

思路：对于f(n)，如果第一次跳1阶，那么剩下f(n-1)，如果第一次跳2阶，那么剩下f(n-2)……如果第一次跳n阶，那么剩下f(0)

那么：f(n) = f(n-1) + f(n-2) + …… + f(1)，同时 f(n-1) = f(n-2) + f(n-3) + …… +f(1)，那么f(n) = 2*f(n-1)

	public class Solution {
	    public int JumpFloorII(int target) {
	        if(target == 0){
	            return 0;
	        }
	        if(target == 1){
	            return 1;
	        }
	        return 2*JumpFloorII(target-1);
	    }
	}

**10、题目描述**

我们可以用2*1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2*1的小矩形无重叠地覆盖一个2*n的大矩形，总共有多少种方法？

	public class Solution {
	    public int RectCover(int target) {
	        int n = target;
	        if(n == 0){
	            return 0;
	        }
	        if(n == 1){
	            return 1;
	        }
	        if(n == 2){
	            return 2;
	        }
	        return RectCover(n-1) + RectCover(n-2);
	    }
	}


**11、题目描述**

输入一个整数，输出该数二进制表示中1的个数。其中负数用补码表示。

	public class Solution {
	    public int NumberOf1(int n) {
	        if(n == 0){
	            return 0;
	        }
	        int num = Math.abs(n);
	        String s = "";
	        while(num/2 != 0){
	            if(num % 2 == 0){
	                s = "0" + s;
	            }else{
	                s = "1" + s;
	            }
	            num /= 2;
	        }
	        s = "1"+s;
	        int count = 0;
	        if(n > 0){
	            for(int i=0; i<s.length(); i++){
	                if(s.charAt(i) == '1'){
	                    count++;
	                }
	            }
	        } else {
	            int index = 0;
	            for(int i=s.length()-1; i>=0; i--){
	                if(s.charAt(i) == '1'){
	                    index = i;
	                    break;
	                }
	            }
	            for(int i=0; i<index; i++){
	                if(s.charAt(i) == '0'){
	                    count++;
	                }
	            }
	            count = count + 1 + 32 - s.length();
	        }
	        return count;
	    }
	    
	}

**12、题目描述**

输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。

	public class Solution {
	    public void reOrderArray(int [] array) {
	        if(array == null || array.length==0){
	            return;
	        }
	        int[] arr = array;
	        int[] newArr = new int[arr.length];
	        int k = 0;
	        for(int i=0; i<arr.length; i++){
	            if(arr[i]%2 == 1){
	                newArr[k] = arr[i];
	                k++;
	            }
	        }
	        for(int i=0; i<arr.length; i++){
	            if(arr[i]%2 == 0){
	                newArr[k] = arr[i];
	                k++;
	            }
	        }
	        for(int i=0; i<newArr.length; i++){
	            array[i] = newArr[i];
	        }
	    }
	}

**13、题目描述**

输入一个链表，输出该链表中倒数第k个结点。

	/*
	public class ListNode {
	    int val;
	    ListNode next = null;
	
	    ListNode(int val) {
	        this.val = val;
	    }
	}*/
	public class Solution {
	    public ListNode FindKthToTail(ListNode head,int k) {
	        if(k <= 0 || head==null){
	            return null;
	        }
	        ListNode p = head; 
	        ListNode q = head;
	        for(int i=1; i<k; i++){
	            if(q.next != null){
	                q = q.next;
	            }else{
	                return null;
	            }
	        }
	        while(q.next != null){
	            q = q.next;
	            p = p.next;
	        }
	        return p;
	    }
	}

**20、题目描述**

定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））。
	
	import java.util.Stack;
	
	public class Solution {
	
	    Stack<Integer> s = new Stack<Integer>();
	    Stack<Integer> s2 = new Stack<Integer>();
	    public void push(int node) {
	        if(s2.isEmpty()){
	            s2.push(node);
	        }else if(node <= s2.peek()){
	            s2.push(node);
	        }
	        s.push(node);
	    }
	    
	    public void pop() {
	        if(s.peek() == s2.peek()){
	            s2.pop();
	        }
	        s.pop();
	    }
	    
	    public int top() {
	        return s.peek();
	    }
	    
	    public int min() {
	        return s2.peek();
	    }
	}

**21、题目描述**

输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）
	
	import java.util.ArrayList;
	
	public class Solution {
	    public boolean IsPopOrder(int [] pushA,int [] popA) {
	      ArrayList<Integer> list = new ArrayList<>();
	        int[] book = new int[pushA.length];
	        for(int i=0; i<popA.length; i++){
	            if(list.contains(popA[i])){
	                if(list.get(list.size()-1) == popA[i]){
	                    list.remove(list.size()-1);
	                    continue;
	                }else{
	                    return false;
	                }
	            }else{
	                for(int j=0; j<pushA.length; j++){
	                if(pushA[j] != popA[i] && book[j]==0){
	                    book[j] = 1;
	                    list.add(pushA[j]);
	                } else{
	                    book[j] = 1;
	                    break;
	                }
	            }
	            }
	            
	        }
	        if(list.size() == 0){
	            return true;
	        }else{
	            return false;
	        }
	        
	    }
	}


**22、题目描述**

从上往下打印出二叉树的每个节点，同层节点从左至右打印。

	import java.util.ArrayList;
	/**
	public class TreeNode {
	    int val = 0;
	    TreeNode left = null;
	    TreeNode right = null;
	
	    public TreeNode(int val) {
	        this.val = val;
	
	    }
	
	}
	*/
	public class Solution {
	    public ArrayList<Integer> PrintFromTopToBottom(TreeNode root) {
	        ArrayList<TreeNode> list = new ArrayList<>();
	        ArrayList<Integer> result = new ArrayList<>();
	        if(root == null){
	            return result;
	        }
	        int head=0, tail=0;
	        list.add(root);
	        tail++;
	        while(head < tail){
	            TreeNode left = list.get(head).left;
	            TreeNode right = list.get(head).right;
	            if(left != null){
	                list.add(left);
	                tail++;
	            }
	            if(right != null){
	                list.add(right);
	                tail++;
	            }
	            head++;
	        }
	        for(TreeNode node : list){
	            result.add(node.val);
	        }
	        return result;
	    }
	}

**22、题目描述**

输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则输出Yes,否则输出No。假设输入的数组的任意两个数字都互不相同。
	
	public class Solution {
	    public boolean VerifySquenceOfBST(int [] sequence) {
	        int[] a = sequence;
	        if(a==null || a.length==0){
	            return false;
	        }
	        return judge(a, 0, a.length-1);
	    }
	    private boolean judge(int[] a, int l, int r){
	        if(l>=r){
	            return true;
	        }
	        int i = r;
	        while(i>l && a[i-1]>a[r]){
	            i--;
	        }
	        for(int j=l; j<i; j++){
	            if(a[j] > a[r]){
	                return false;
	            }
	        }
	        return judge(a, l, i-1) && judge(a, i, r-1);
	    }
	}

**23、题目描述**

输入一颗二叉树的跟节点和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。(注意: 在返回值的list中，数组长度大的数组靠前)
	
	import java.util.ArrayList;
	/**
	public class TreeNode {
	    int val = 0;
	    TreeNode left = null;
	    TreeNode right = null;
	
	    public TreeNode(int val) {
	        this.val = val;
	
	    }
	
	}
	*/
	public class Solution {
	    public ArrayList<ArrayList<Integer>> paths = new ArrayList<ArrayList<Integer>>();
	    public ArrayList<Integer> path = new ArrayList<Integer>();
	    public ArrayList<ArrayList<Integer>> FindPath(TreeNode root,int target) {
	        if(root == null){
	            return paths;
	        }
	        path.add(root.val);
	        if(root.left==null && root.right == null){
	            if(target-root.val == 0){
	                paths.add(new ArrayList<Integer>(path));
	            }
	        }
	        
	        FindPath(root.left, target-root.val);
	        FindPath(root.right, target-root.val);
	        path.remove(path.size()-1);
	        return paths;
	    }
	}

**24、题目描述**

输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针指向任意一个节点），返回结果为复制后复杂链表的head。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）

	/*
	public class RandomListNode {
	    int label;
	    RandomListNode next = null;
	    RandomListNode random = null;
	
	    RandomListNode(int label) {
	        this.label = label;
	    }
	}
	*/
	public class Solution {
	    public RandomListNode Clone(RandomListNode pHead)
	    {
	        if(pHead == null){
	            return null;
	        }
	        //1.不考虑random指针，复制每一个节点插入到原节点后面
	        //变成了A-->A'-->B-->B'-->C-->C'
	        RandomListNode cur = pHead;
	        while(cur != null){
	            RandomListNode curClone = new RandomListNode(cur.label);
	            RandomListNode curNext = cur.next;
	            curClone.next = curNext;
	            cur.next = curClone;
	            cur = curNext;
	        }
	        //2.处理random指针
	        //如果A的random指针是C，那么A'的random指针应该是A.random.next，也就是C'
	        cur = pHead;
	        while(cur != null){
	            RandomListNode next = cur.next;
	            if(cur.random != null){
	                next.random = cur.random.next;
	            }
	            cur = next.next;
	        }
	        //3.拆分整个链表，也不需要再考虑random指针
	        cur = pHead;
	        RandomListNode cHead = cur.next;
	        while(cur.next != null){
	            RandomListNode curNext = cur.next;
	            cur.next = curNext.next;
	            cur = curNext;
	        }
	        return cHead;
	    }
	}

**25、题目描述**

输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的结点，只能调整树中结点指针的指向。

	/**
	public class TreeNode {
	    int val = 0;
	    TreeNode left = null;
	    TreeNode right = null;
	
	    public TreeNode(int val) {
	        this.val = val;
	
	    }
	
	}
	*/
	import java.util.Stack;
	public class Solution {
	    public TreeNode Convert(TreeNode pRootOfTree) {
	        if(pRootOfTree == null){
	            return null;
	        }
	        //核心思想是二叉树的中序遍历
	        Stack<TreeNode> stack = new Stack<>();
	        TreeNode p = pRootOfTree;
	        TreeNode pre = null;
	        boolean isFirst = true; //双向链表的头结点
	        while(p!=null || !stack.isEmpty()){
	            while(p!=null){
	                stack.push(p);
	                p = p.left;
	            }
	            p = stack.pop();
	            if(isFirst){
	                pRootOfTree = p;
	                pre = p;
	                isFirst = false;
	            }else{
	                pre.right = p;
	                p.left = pre;
	                pre = p;
	            }
	            p = p.right;
	        }
	        return pRootOfTree;
	    }
	}

**26、题目描述**

输入一个字符串,按字典序打印出该字符串中字符的所有排列。例如输入字符串abc,则打印出由字符a,b,c所能排列出来的所有字符串abc,acb,bac,bca,cab和cba。

	import java.util.ArrayList;
	import java.util.Collections;
	public class Solution {
	    public ArrayList<String> Permutation(String str) {
	        ArrayList<String> list = new ArrayList<String>();
	        PermutationHelper(str.toCharArray(), 0, list);
	        Collections.sort(list);
	        return list;
	    }
	    
	    private void PermutationHelper(char[] chs, int start, ArrayList<String> list){
	        if(start==chs.length-1)
	        {
	            String s = "";
	            for(int i=0; i<chs.length; i++){
	                s += chs[i];
	            }
	            if(!list.contains(s)){
	                list.add(s);
	            }
	            
	            //如果已经到了数组的最后一个元素，前面的元素已经排好，输出。
	        }
	        for(int i=start;i<=chs.length-1;i++)
	        {
	        //把第一个元素分别与后面的元素进行交换，递归的调用其子数组进行排序
	                Swap(chs,i,start);
	                PermutationHelper(chs,start+1, list);
	                Swap(chs,i,start);
	        //子数组排序返回后要将第一个元素交换回来。  
	        //如果不交换回来会出错，比如说第一次1、2交换，第一个位置为2，子数组排序返回后如果不将1、2
	        //交换回来第二次交换的时候就会将2、3交换，因此必须将1、2交换使1还是在第一个位置 
	        }
	    }
	     private  void Swap(char chs[],int i,int j)
	    {
	        char temp;
	        temp=chs[i];
	        chs[i]=chs[j];
	        chs[j]=temp;
	    }
	}

**27、题目描述**

数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0。

	import java.util.Arrays;
	public class Solution {
	    public int MoreThanHalfNum_Solution(int [] array) {
	        if(array==null || array.length==0){
	            return 0;
	        }
	        int len = array.length;
	        int[] book = new int[len];
	        Arrays.fill(book, 1);
	        for(int i=0; i<len-1; i++){
	            for(int j=i+1; j<len; j++){
	                if(array[j] == array[i]){
	                    book[i]++;
	                }
	            }
	        }
	        int max = 0;
	        int ind = 0;
	        for(int i=0; i<len; i++){
	            if(book[i] > max){
	                max = book[i];
	                ind = i;
	            }
	        }
	        if(max > len>>1){
	            return array[ind];
	        }else{
	            return 0;
	        }
	    }
	}

**28、题目描述**

输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。

	import java.util.Arrays;
	import java.util.ArrayList;
	public class Solution {
	    public ArrayList<Integer> GetLeastNumbers_Solution(int [] input, int k) {
	        ArrayList<Integer> list = new ArrayList<Integer>();
	        if(input==null || input.length==0 || k<=0 || k>input.length){
	            return list;
	        }
	        Arrays.sort(input);
	        for(int i=0; i<k; i++){
	            list.add(input[i]);
	        }
	        return list;
	    }
	}

**29、题目描述**

HZ偶尔会拿些专业问题来忽悠那些非计算机专业的同学。今天测试组开完会后,他又发话了:在古老的一维模式识别中,常常需要计算连续子向量的最大和,当向量全为正数的时候,问题很好解决。但是,如果向量中包含负数,是否应该包含某个负数,并期望旁边的正数会弥补它呢？例如:{6,-3,-2,7,-15,1,2,2},连续子向量的最大和为8(从第0个开始,到第3个为止)。给一个数组，返回它的最大连续子序列的和，你会不会被他忽悠住？(子向量的长度至少是1)
	
	public class Solution {
	    public int FindGreatestSumOfSubArray(int[] array) {
	        if(array==null || array.length==0){
	            return 0;
	        }
	        int max = Integer.MIN_VALUE;
	        int len = array.length;
	        int[][] map = new int[len][len];
	        for(int i=0; i<len; i++){
	            map[i][i] = array[i];
	            if(array[i] > max){
	                max = array[i];
	            }
	        }
	        for(int i=0; i<len; i++){
	            for(int j=i+1; j<len; j++){
	                map[i][j] = map[i][j-1]+array[j];
	                if(map[i][j] > max){
	                    max = map[i][j];
	                }
	            }
	        }
	        return max;
	    }
	}

**30、题目描述**

输入一个正整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。例如输入数组{3，32，321}，则打印出这三个数字能排成的最小数字为321323。

	import java.util.Arrays;
	
	public class Solution {
	    public String PrintMinNumber(int [] numbers) {
	        if(numbers==null || numbers.length==0){
	            return "";
	        }
	        int len = numbers.length;
	        int[] book = new int[len];
	        String[] nums = new String[len];
	        int max = 0;
	        for(int i=0; i<len; i++){
	            if(numbers[i] > max){
	                max = numbers[i];
	            }
	        }
	        int maxLen = (max+"").length();
	        //把所有的数字变成等长的字符串，不足的字符串用每个字符串的首字符填充
	        //3   32  321
	        //333 323 321
	        //然后按字符串从小到大排序即可
	        for(int i=0; i<len; i++){
	            String s = numbers[i]+"";
	            while(s.length() < maxLen){
	                s += s.charAt(0);
	            }
	            nums[i] = s;
	        }
	        for(int i=0; i<nums.length-1; i++){
	            for(int j=nums.length-1; j>i; j--){
	                if(nums[j].compareTo(nums[j-1]) < 0){
	                    String t = nums[j];
	                    nums[j] = nums[j-1];
	                    nums[j-1] = t;
	                    int tmp = numbers[j];
	                    numbers[j] = numbers[j-1];
	                    numbers[j-1] = tmp;
	                }
	            }
	        }
	        String res = "";
	        for(int i=0; i<len; i++){
	            res += numbers[i];
	        }
	        return res;
	    }
	}