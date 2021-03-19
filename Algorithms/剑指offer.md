# 剑指offer

### 03.数组中重复的数字

直接用HashSet存储遍历过的元素即可。

```
class Solution {
    public int findRepeatNumber(int[] nums) {
        Set<Integer> set = new HashSet<>();
        for (int i : nums) {
            if (set.contains(i)) {
                return i;
            }
            set.add(i);
        }
        return -1;
    }
}
```

* 方法二：鸽笼原理

  由于题目中说了，n个数字范围0~n-1，所以如果没有重复，则必然数字和index一一对应，那么可以遍历数组，拿到数字后与对应index中的数字进行置换，直到遇到重复数字。  

```
  class Solution {
      public int findRepeatNumber(int[] nums) {
          for (int i = 0; i < nums.length; i++) {
              while (nums[i] != i) {
                  if (nums[i] == nums[nums[i]]) {
                      return nums[i];
                  }
                  int tmp = nums[nums[i]];
                  nums[nums[i]] = nums[i];
                  nums[i] = tmp;
              }
          }
          return -1;
      }
  }
```
### 04.二维数组中的查找
对于这个二维数组中任意元素，其右下一定比它大，左上一定比它小。  
从左下角开始找，如果比他小，则排除当前行，向上找；如果比他大，则排除当前列，向右找。  
为什么从左下角？因为这样好控制范围，i--可以删除最后一行，j++可以删除左边一列。  
```
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        if (matrix.length == 0 || matrix[0].length == 0) return false;
        int n = matrix.length;
        int m = matrix[0].length;
        int j = 0;
        int i = n - 1;

        while (j < m && i >= 0) {
            if (matrix[i][j] > target) i--;
            else if (matrix[i][j] < target) j++;
            else return true;
        }
        return false;
        
    }
}
```
### 05.替换空格
直接遍历即可
```
class Solution {
    public String replaceSpace(String s) {
        if (s.length() == 0) return s;
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == ' ') {
                sb.append("%20");
            } else {
                sb.append(s.charAt(i));
            }
        }
        return sb.toString();
    }
}
```
### 06.从尾到头打印链表
看到反转，想到栈
```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public int[] reversePrint(ListNode head) {
        Stack<ListNode> stack = new Stack<>();
        ListNode temp = head;
        while (temp != null) {
            stack.push(temp);
            temp = temp.next;
        }
        int size = stack.size();
        int[] arr = new int[size];
        for (int i = 0; i < size; i++) {
            arr[i] = stack.pop().val;
        }
        return arr;
    }
}
```
### 07.重建二叉树
分治法，找到root，然后再中序里分开左右子树，然后递归  
```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    int[] preorder;
    int[] inorder;
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        if (preorder.length == 0) return null;
        this.preorder = preorder;
        this.inorder = inorder;
        TreeNode root = buildTree(0, preorder.length - 1, 0, preorder.length - 1);
        return root;
    }

    private TreeNode buildTree(int i, int j, int x, int y) {
        if (i > j || x > y) return null;
        // System.out.println(i + " " + j + " " + x + " " + y);
        TreeNode root = new TreeNode(preorder[i]);
        int a = x;
        for (; a < y; a++) {
            if (inorder[a] == preorder[i]) {
                break;
            }
        }
        int leftLen = a - x;
        int rightLen = y - a;
        root.left = buildTree(i + 1, i + leftLen, x, a - 1);
        root.right = buildTree(i + leftLen + 1, j, a + 1, y);
        return root;
    }
}
```
### 09.用两个栈实现队列
```
class CQueue {
    private LinkedList<Integer> stack1;
    private LinkedList<Integer> stack2;

    public CQueue() {
        stack1 = new LinkedList<>();
        stack2 = new LinkedList<>();
    }
    
    public void appendTail(int value) {
        stack2.add(value);
    }
    
    public int deleteHead() {
        if (!stack1.isEmpty()) {
            return stack1.removeFirst();
        }
        if (stack2.isEmpty()) {
            return -1;
        }
        while (!stack2.isEmpty()) {
            stack1.add(stack2.removeFirst());
        }
        return stack1.removeFirst();
    }
}

/**
 * Your CQueue object will be instantiated and called as such:
 * CQueue obj = new CQueue();
 * obj.appendTail(value);
 * int param_2 = obj.deleteHead();
 */
```
### 10-I.斐波那契数列
简单的动态规划，按照状态方程来写即可
```
F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
```
```
class Solution {
    public int fib(int n) {
        if (n < 2) return n;
        int[] f = new int[n + 1];
        f[0] = 0;
        f[1] = 1;
        for (int i = 2; i < n + 1; i++) {
            f[i] = (int)((f[i - 1] + f[i - 2]) % (1e9+7));
            // System.out.println(Arrays.toString(f));
        }
        return f[n];
    }
}
```
### 10-II.青蛙跳台阶问题
动态规划，状态方程和斐波那契一样
```
class Solution {
    public int numWays(int n) {
        if (n == 0) return 1;
        if (n == 1) return 1;
        int[] dp = new int[n + 1];
        dp[0] = 1;
        dp[1] = 1;
        dp[2] = 2;
        for (int i = 3; i < n + 1; i++) {
            dp[i] = (int)((dp[i - 1] + dp[i - 2]) % (1e9+7));
        }
        return dp[n];
    }
}
```
### 11.旋转数组的最小数字
记录上一个数字，找到第一个前比后大的，如果找不到，说明没旋转，则第一个数字是最小的  
```
class Solution {
    public int minArray(int[] numbers) {
        int last = Integer.MIN_VALUE;
        
        for (int i : numbers) {
            if (last > i) {
                return i;
            }
            last = i;
        }
        return numbers[0];
    }
}
```
###  12. 矩阵中的路径
深度优先搜索，board中搜索过的区域标记一下，弹栈的时候再把这个标记变回原来的字母  
```
class Solution {
    public boolean exist(char[][] board, String word) {
        if (board.length == 0 || board[0].length == 0) {
            return false;
        }
        char[] words = word.toCharArray();

        for (int i = 0; i < board.length; i++) {
            for (int j = 0; j < board[0].length; j++) {
                if (dfs(board, words, i, j, 0)) {
                    return true;
                }
            }
        }
        return false;
    }

    private boolean dfs(char[][] board, char[] words, int i, int j, int k) {
        if (i < 0 || i >= board.length || j < 0 || j >= board[0].length || words[k] != board[i][j]) {
            return false;
        } 

        if(k == words.length - 1) return true;

        board[i][j] = '\0';
        boolean res = dfs(board, words, i + 1, j, k + 1) || dfs(board, words, i - 1, j, k + 1) || 
                      dfs(board, words, i, j + 1, k + 1) || dfs(board, words, i , j - 1, k + 1);
        board[i][j] = words[k];
        return res;
    }
}
```
### 机器人的运动范围
从左上角开始搜索，递归搜索每一个位置的下和右  
```
class Solution {
    public int movingCount(int m, int n, int k) {
        if (m == 0 || n == 0) {
            return 0;
        }
        boolean[][] used = new boolean[m][n];
        
        return dfs(m, n, 0, 0, k, used);
    }

    private int dfs(int m, int n, int i, int j, int k, boolean[][] used) {
        if (i >= m || j >= n || (bitsum(i) + bitsum(j)) > k || used[i][j]) {
            return 0;
        }
    
        used[i][j] = true;
        return 1 + dfs(m, n, i + 1, j, k, used) + dfs(m, n, i, j + 1, k, used);
    }

    private int bitsum(int i) {
        int sum = 0;
        while (i > 0) {
            sum += i % 10;
            i /= 10;
        }
        return sum;
    }
}
```
### 14-I.剪绳子
dfs，每次剪一段，遍历这一段的长度，然后搜索两种情况：后面剩下的绳子剪还是不剪。  
注意要记忆化，因为做了大量重复搜索。  
memo数组保存当长度为i时，可以得到的最大值  
```
class Solution {
    int[] memo = new int[58];
    public int cuttingRope(int n) {
        if (n == 2) {
            return 1;
        }

        int res = 0;
        // 遍历第一段可以截开的长度
        for (int i = 2; i < n; i++) {
            int next = 0;
            if (memo[n - i] != 0) {
                next = memo[n - i];
            } else {
                next = cuttingRope(n - i);
                memo[n - i] = cuttingRope(n - i);
            }
            res = Math.max(res, Math.max(i * next, i * (n - i)));

        }
        return res;
    
    }
}
```
### 14-II.剪绳子II
两个推论：
1. 分割成长度相等的段，乘积最大
2. 分割成长度接近3的段，乘积最大
```
class Solution {
    public int cuttingRope(int n) {
        if (n == 2) {
            return 1;
        }
        if (n == 3) {
            return 2;
        }
        long res = 1;
        while (n > 4) {
            res *= 3;
            res %= 1000000007;
            n -= 3;
        }
        res *= n;
        return (int)(res % 1000000007);
    }   
}
```
### 15.二进制中1的个数
注意都转成正数做，负数要异或一下10000000000000000000000000000000，也就是Integer.MIN_VALUE  
```
public class Solution {
    // you need to treat n as an unsigned value
    public int hammingWeight(int n) {
        if (n == 0) return 0;
        int res = n > 0 ? 0 : 1;
        if (n < 0) {
            n ^= Integer.MIN_VALUE;
        }
        while (n > 0) {
            res += n & 1;
            n >>= 1;
        }
        return res;
    }
}
```
### 16. 数值的整数次方
**快速幂**，x^n中，n转化为二进制为bm ...b3 b2 b1  
则二进制转十进制为n = 1b1 + 2b2 + 4b3 + … + 2^(m-1)bm  
则x^n = x^(1b1 + 2b2 + 4b3 + … + 2^(m-1)bm) = (x^b1)\*(x^2b2)\*(x^4b3)\* … \*(x^2^(m-1)bm)  
也就是利用二分法将幂次计算中的乘法复杂度从O(n)降到了O(logn)  

```
class Solution {
    public double myPow(double x, int n) {
        // 二分思想，快速幂
        if (x == 0) return 0;
        long b = n;

        if(b < 0) {
            x = 1 / x;
            b = -b;
        }
        double res = 1.0;
        // 每次乘的幂次都高2
        while (b > 0) {
            if ((b & 1) == 1) res *= x;
            x *= x;
            b >>= 1; 
        }
        return res;
    }
}
```
### 17.打印从1到最大的n位数
总数是Math.pow(10, n) - 1  
```
class Solution {
    public int[] printNumbers(int n) {
        int count = (int)Math.pow(10, n) - 1;
        int[] res = new int[count];
        for (int i = 1; i < count + 1; i++) {
            res[i - 1] = i;
        }
        return res;
    }
}
```
### 18.删除链表的节点
如果head需要删除，返回head.next  
删除一个中间节点需要拿到被删除节点的上一个节点，与被删除节点的下一个节点连接起来即可  
```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode deleteNode(ListNode head, int val) {
        ListNode ptr = head;
        if (ptr.val == val) return ptr.next;
        while (ptr.next != null && ptr.next.val != val) {
            ptr = ptr.next;
        }
        ptr.next = ptr.next.next;
        return head;
    }
}
```
### 19. 正则表达式匹配
动态规划，定义一个dp\[n + 1\]\[m + 1\]，n是s的长度，m是p的长度。dp\[i\]\[j\]==true代表s[:i-1]==p[:j-1]  
看题解https://leetcode-cn.com/problems/zheng-ze-biao-da-shi-pi-pei-lcof/solution/jian-zhi-offer-19-zheng-ze-biao-da-shi-pi-pei-dong/  
```
class Solution {
    public boolean isMatch(String s, String p) {
        int n = s.length();
        int m = p.length();
        boolean[][] dp = new boolean[n + 1][m + 1];
        dp[0][0] = true;
        // 如果s长度为0，则需要让p中偶数位都为‘*’，奇数位出现0次
        for (int i = 2; i < m + 1; i += 2) {
            dp[0][i] = dp[0][i - 2] && p.charAt(i - 1) == '*';
        }
        for (int i = 1; i < n + 1; i++) {
            for (int j = 1; j < m + 1; j++) {
                if (p.charAt(j - 1) == '*') {
                    // 看作出现0次
                    if (dp[i][j - 2]) dp[i][j] = true;
                    // 看作出现1次
                    else if (dp[i][j - 1]) dp[i][j] = true;
                    // 看作多出现了一次，即前面已经有*匹配成功了，s又多出了一个对应字符
                    else if (dp[i - 1][j] && s.charAt(i - 1) == p.charAt(j - 2)) dp[i][j] = true;
                    // 看作'.'多出现了一次
                    else if (dp[i - 1][j] && p.charAt(j - 2) == '.') dp[i][j] = true;

                } else {
                    if (dp[i - 1][j - 1] && s.charAt(i - 1) == p.charAt(j - 1)) dp[i][j] = true;
                    if (dp[i - 1][j - 1] && p.charAt(j - 1) == '.') dp[i][j] = true;
                }
            }
        }
        return dp[n][m];
    }
}

```
### 20.表示数值的字符串
按照出现情况顺序判断，这题思路比较巧  
https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/solution/javaban-ben-ti-jie-luo-ji-qing-xi-by-yangshyu6/  
```
class Solution {
    public boolean isNumber(String s) {
        if (s == null || s.length() == 0) {
            return false;
        }

        char[] ss = s.trim().toCharArray();
        int len = ss.length;

        boolean dot = false;
        boolean e = false;
        boolean num = false;

        for (int i = 0; i < len; i++) {
            if (ss[i] >= '0' && ss[i] <= '9') {
                num = true;
            } else if (ss[i] == '.') {
                if (dot || e) {
                    return false;
                }
                dot = true;
            } else if (ss[i] == 'e' || ss[i] == 'E') {
                if (e || !num) {
                    return false;
                }
                e = true;
                //重置num，排除123e或者123e+的情况,确保e之后也出现数
                num = false;
            } else if (ss[i] == '+' || ss[i] == '-') {
                //+-出现在0位置或者e/E的后面第一个位置才是合法的
                if(i != 0 && ss[i-1] != 'e' && ss[i-1] != 'E'){
                    return false;
                }
            } else {
                return false;
            }
        }
        return num;
    }
}
```
### 21. 调整数组顺序使奇数位于偶数前面
双指针，遇到偶数就往数组最后换，直到两个指针碰上  
```
class Solution {
    public int[] exchange(int[] nums) {
        int back = nums.length - 1;
        int ptr = 0;
        
        while (ptr < back) {
            if (nums[ptr] % 2 == 0) {
                int tmp = nums[back];
                nums[back--] = nums[ptr];
                nums[ptr] = tmp;        
            } else {
                ptr++;
            }
        }
        return nums;
    }
}
```
### 22.链表中倒数第k个节点
* 方法一：遍历两遍，第一遍数数量，第二遍找节点  
```
class Solution {
    public ListNode getKthFromEnd(ListNode head, int k) {
        if (head == null) return null;
        int count = 0;
        ListNode ptr = head;
        while (ptr != null) {
            count++;
            ptr = ptr.next;
        }
        count -= k;

        while (count > 0) {
            head = head.next;
            count--;
        }
        return head;
    }
}
```

* **方法二：双指针，先让一个指针走k步，然后两个指针一起走，后一个指针走到头的时候第一个指针即为目标节点** 
```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode getKthFromEnd(ListNode head, int k) {
        if (head == null) return null;
        
        ListNode ptr = head;
        ListNode ptr2 = head;
        while (k > 0) {
            ptr2 = ptr2.next;
            k--;
        }

        while (ptr2 != null) {
            ptr2 = ptr2.next;
            ptr = ptr.next;
        }
        return ptr;
    }
}
```

### 24.反转链表
* 方法一：开辟一个新链表，头插法遍历一遍即可  
```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode root = new ListNode(1);
        ListNode ptr = root;
        while (head != null) {
            ListNode tmp = ptr.next;
            ptr.next = head;
            ListNode tmp2 = head.next;
            head.next = tmp;
            head = tmp2;
        }
        return root.next;
    }
}
```
* 方法二：原地反转，双指针，不断让后一个节点反向指向前一个
```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {
        if (head == null) return null;
        ListNode cur = null;
        ListNode pre = head;
        while (pre != null) {
            ListNode tmp = pre.next;
            pre.next = cur;
            cur = pre;
            pre = tmp;
        }
        return cur;
    }
}
```
### 25. 合并两个排序的链表
开一个新链表合并即可,注意l2为空时的判断
```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode res = new ListNode(1);
        ListNode ptr = res;
        while (l1 != null || l2 != null) {
            if (l2 == null || l1 != null && l1.val <= l2.val) {
                ptr.next = l1;
                l1 = l1.next;
            } else {
                ptr.next = l2;
                l2 = l2.next;
            }
            ptr = ptr.next;
        }
        return res.next;
    }   
}
```
### 26.树的子结构
两棵树相等的条件：
* 两棵树都不为null
* 每个节点的值相同
* 每个节点的子树相同（子树可以为null）
```
class Solution {
    public boolean isSubStructure(TreeNode A, TreeNode B) {
        return (A != null && B != null) && (dfs(A, B) || isSubStructure(A.left, B) || isSubStructure(A.right, B));
    }

    private boolean dfs(TreeNode A, TreeNode B) {
        // 如果子树B为null，不管A是什么都为true
        if (B == null) return true;

        // 如果A为null而B不为null，或者值不相同，false
        if (A == null || A.val != B.val) return false;

        // 如果这个节点判定相同，分别递归左右子树
        return dfs(A.left, B.left) && dfs(A.right, B.right);
    }
}
```
### 27.二叉树的镜像
递归反转左右子树即可
```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode mirrorTree(TreeNode root) {
        if (root == null) return null;
        TreeNode tmp = root.left;
        root.left = root.right;
        root.right = tmp;

        mirrorTree(root.left);
        mirrorTree(root.right);
        return root;
    }
}
```
### 28.对称的二叉树
左右子树分别递归，拿左左和右右比，左右和右左比
```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public boolean isSymmetric(TreeNode root) {
        if (root == null) return true;
        
        return dfs(root.left, root.right);
    }

    public boolean dfs(TreeNode A, TreeNode B) {
        if (A == null && B == null) {
            return true;
        }

        if (A == null || B == null) {
            return false;
        }

        if (A.val != B.val) return false;

        return dfs(A.left, B.right) && dfs(A.right, B.left);
    }
}
```
### 29.顺时针打印矩阵
收缩法，定义四个边界，每打印一条边就收缩一个边界  
```
class Solution {
    public int[] spiralOrder(int[][] matrix) {
        if (matrix.length == 0 || matrix[0].length == 0) return new int[0];

        int a = 0, b = matrix.length;
        int x = 0, y = matrix[0].length;
        int count = 0;
        int[] res = new int[(b) * (y)];
        b--;
        y--;
        while (a <= b && x <= y) {
            for (int i = x; i <= y; i++) {
                res[count++] = matrix[a][i];
            }
            a++;
            if (count >= res.length) break;

            for (int i = a; i <= b; i++) {
                res[count++] = matrix[i][y];
            }
            y--;
            if (count >= res.length) break;

            for (int i = y; i >= x; i--) {
                res[count++] = matrix[b][i];

            }
            b--;
            if (count >= res.length) break;

            for (int i = b; i >= a; i--) {
                res[count++] = matrix[i][x];

            }
            x++;
            if (count >= res.length) break;

        }
        return res;

    }
}
```
### 30. 包含min函数的栈
定义两个栈，stack1存储所有数据，stack2只存储当前最小的数字。  
当压栈时，只有压栈的数字小于stack2栈顶时才压入。  
```
class MinStack {

    LinkedList<Integer> stack1;
    LinkedList<Integer> stack2;

    /** initialize your data structure here. */
    public MinStack() {
        stack1 = new LinkedList<>();
        stack2 = new LinkedList<>();
    }
    
    public void push(int x) {
        stack1.add(x);
        if (stack2.isEmpty() || x <= stack2.getLast()) {
            stack2.add(x);
        }
    }
    
    public void pop() {
        int x = stack1.removeLast();
        if (x == stack2.getLast()) {
            stack2.removeLast();
        }
    }
    
    public int top() {
        return stack1.getLast();
    }
    
    public int min() {
        return stack2.getLast();
    }
}

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack obj = new MinStack();
 * obj.push(x);
 * obj.pop();
 * int param_3 = obj.top();
 * int param_4 = obj.min();
 */
```
### 31. 栈的压入、弹出序列
模拟法，开一个栈，每次压入一个元素，然后循环判断栈顶是否等于popped遍历到的那个元素，如果等于，则弹栈。最后压栈的元素遍历完，如果栈弹空了，则表示true  
```
class Solution {
    public boolean validateStackSequences(int[] pushed, int[] popped) {
        if (pushed.length != popped.length) return false;
        int len = pushed.length;
        if (len == 0) return true;
        
        LinkedList<Integer> list = new LinkedList<>();

        int cur = 0;
        for (int i = 0; i < len; i++) {
            list.add(pushed[i]);

            while (!list.isEmpty() && list.getLast() == popped[cur]) {
                list.removeLast();
                cur++;
            }
        }
        return list.isEmpty();
    }
}
```
### 32 - I. 从上到下打印二叉树
层序遍历二叉树，用队列Queue来做。  
每遍历到一个节点，就把它的值加入结果，并把它的左右孩子加入队列。  
```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public int[] levelOrder(TreeNode root) {
        Queue<TreeNode> queue = new LinkedList<>();
        List<Integer> list = new ArrayList<>();

        if (root == null) return new int[0];
        queue.offer(root);
        
        while (!queue.isEmpty()) {
            TreeNode node = queue.poll();
            list.add(node.val);
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right); 
        }
        int[] res = new int[list.size()];
        for (int i = 0; i < list.size(); i++) {
            res[i] = list.get(i);
        }
        return res;
    }
}
```
### 32 - II. 从上到下打印二叉树 II
跟上题差不多，层序遍历。不过这题要求每层分别打印，那么就维持int值，代表每层有多长，作为分层的界限即可  
```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        Queue<TreeNode> queue = new LinkedList<>();
        // 记录上一层的长度
        int preLen = 0;
        // 记录新层的长度
        int newLen = 0;

        if (root == null) return res;
        queue.offer(root);
        preLen = 1;

        while (!queue.isEmpty()) {
            List<Integer> list = new ArrayList<>();
            while (preLen > 0) {
                TreeNode node = queue.poll();
                preLen--;
                list.add(node.val);
                if (node.left != null) {
                    queue.add(node.left);
                    newLen++;
                }
                if (node.right != null) {
                    queue.add(node.right);
                    newLen++;
                }
            }   
            res.add(list);
            preLen = newLen;
            newLen = 0;
        }
        return res;
    }
}
```
### 32 - III. 从上到下打印二叉树 III
跟上两题也差不多，特殊的是要记录一个flag，标记当前层是正序还是反序  
```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if (root == null) return res;
        LinkedList<TreeNode> list1 = new LinkedList<>();
        LinkedList<TreeNode> list2 = new LinkedList<>();
        // true表示当前行从左向右；false表示当前行从右向左
        boolean flag = true;
        list1.add(root);
        while (!list1.isEmpty()) {
            LinkedList<Integer> tmp = new LinkedList<>();
            while (!list1.isEmpty()) {
                TreeNode node = list1.poll();
                tmp.add(node.val);
                if (node.left != null) list2.offer(node.left);
                if (node.right != null) list2.offer(node.right); 
            }
            if (flag) {
                res.add(tmp);
            } else {
                List<Integer> tmp2 = new ArrayList<>();
                while (!tmp.isEmpty()) {
                    tmp2.add(tmp.removeLast());
                }
                res.add(tmp2);
            }
            flag = !flag;
            list1 = list2;
            list2 = new LinkedList<>();
        }
        return res;
    }
}
```
### 33. 二叉搜索树的后序遍历序列
* 方法一：递归
后序遍历有以下几个特点：
1. 根节点在最后
2. 数组前半段小于root的是左子树，后半段大于root的是右子树
所以可以递归判断是否是后序遍历序列
```
class Solution {
    public boolean verifyPostorder(int[] postorder) {
        int len = postorder.length;
        
        //左开右闭
        return recur(postorder, 0, len);
    }

    private boolean recur(int[] postorder, int i, int j) {
        // 递归跳出
        if (i >= j - 1) return true;
        
        // 后序遍历根节点在最后
        int root = postorder[j - 1];
        // 找到第一个大于root的，就是右子树的第一个节点

        int right = 0;
        for (;right < j - 1; right++) {
            if (postorder[right] > root) {
                break;
            }
        }

        // 遍历右子树是否都大于root
        for (int a = right; a < j - 1; a++) {
            if (postorder[a] <= root) return false;
        }

        // 递归判断左右子树是否是后序遍历序列
        return recur(postorder, i, right) && recur(postorder, right, j - 1);
    }
}
```
* 方法二：**单调栈**
在二叉搜索树中，满足以下几点：
* root的左子树全部都小于root
* root的右子树全部都大于root
* root的右子树全部都大于root的左子树
在二叉搜索树的后序遍历**倒序** [rn, rn-1, ... , r1] 中，满足以下几点：
* 当节点值ri > ri+1时：节点ri一定是ri+1的右子节点
* 当节点值ri < ri+1时：节点ri一定是某个root的左子节点，而且root是节点ri+1,ri+2,...,rn中值大于，且大小最接近ri的节点（这里root是直接连接左子节点ri的）
所以采用**单调栈**方法，倒序遍历，记每个遍历到的节点为ri：
1. 判断：若ri>root，说明此后序遍历序列不满足二叉搜索树定义
2. 更新父节点root：当栈不为空且ri < stack.peek()时（由于前面是递增序列，所以弹出的最后一个节点就是离ri最近的），循环执行出栈，并将出栈节点赋给root
3. 入栈:将当前节点ri入栈
若遍历完成则说明该后序遍历满足二叉搜索树定义，返回true
```
class Solution {
    public boolean verifyPostorder(int[] postorder) {
        int root = Integer.MAX_VALUE;
        Stack<Integer> stack = new Stack<>();

        for (int i = postorder.length - 1; i >= 0; i--) {
            if (postorder[i] > root) return false;
            while (!stack.isEmpty() && postorder[i] < stack.peek()) {
                root = stack.pop();
            }
            stack.push(postorder[i]);
        }
        return true;
    }
}
```
### 34. 二叉树中和为某一值的路径
注意题目中，没有说必须是正整数，且要求路径必须遍历到叶子节点  
直接dfs即可  
```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public List<List<Integer>> pathSum(TreeNode root, int sum) {
        List<List<Integer>> res = new ArrayList<>();
        dfs(res, new ArrayList<Integer>(), root, sum, 0);
        return res;
    }

    private void dfs(List<List<Integer>> res, List<Integer> tmp, TreeNode node, int sum, int cur) {
        if (node == null) return;

        cur += node.val;
        tmp.add(node.val);
        
        if (node.left == null && node.right == null) {
            if (cur == sum) {
                res.add(new ArrayList<>(tmp));
            }
        } 

        dfs(res, tmp, node.left, sum, cur);
        dfs(res, tmp, node.right, sum, cur);

        tmp.remove(tmp.size() - 1);
    }
}
```
### 35. 复杂链表的复制
开辟一个HashMap，存储原链表中的节点对应新链表中的哪个节点

```
/*
// Definition for a Node.
class Node {
    int val;
    Node next;
    Node random;

    public Node(int val) {
        this.val = val;
        this.next = null;
        this.random = null;
    }
}
*/
class Solution {
    public Node copyRandomList(Node head) {
        if (head == null) return null;
        // 开辟一个HashMap，存储原链表中的节点对应新链表中的哪个节点
        Map<Node, Node> map = new HashMap<>();

        Node root = new Node(head.val);
        map.put(head, root);
        Node ptr1 = root;
        Node ptr = head.next;
        // 按照链表复制
        while (ptr != null) {
            ptr1.next = new Node(ptr.val);
            map.put(ptr, ptr1.next);
            ptr = ptr.next;
            ptr1 = ptr1.next;
        }

        // 根据map里保存的节点对应关系，复制random
        ptr1 = root;
        ptr = head;
        while (ptr != null) {
            ptr1.random = map.get(ptr.random);
            ptr1 = ptr1.next;
            ptr = ptr.next;
        }
        return root;
    }
}
```
### 36. 二叉搜索树与双向链表
* **二叉搜索树的中序遍历是一个递增序列**，所以利用中序遍历的过程来构建双向链表
* 中序遍历的过程中构建双向链表
* 先递归构建左子树，记录递归构建之后的最后一个节点为pre，pre的右节点应当为是root，再递归构建右子树
* 最后首尾相连构建循环双向链表
```
/*
// Definition for a Node.
class Node {
    public int val;
    public Node left;
    public Node right;

    public Node() {}

    public Node(int _val) {
        val = _val;
    }

    public Node(int _val,Node _left,Node _right) {
        val = _val;
        left = _left;
        right = _right;
    }
};
*/
class Solution {
    // pre记录cur的左侧节点
    // head用于记录头节点
    Node pre, head;
    public Node treeToDoublyList(Node root) {
        if (root == null) return null;
        // 递归构建双向链表
        dfs(root);
        // 首尾相连
        head.left = pre;
        pre.right = head;
        return head;
        
    }

    private void dfs(Node cur) {
        if (cur == null) {
            return;
        }
        // 先递归左侧，因为左侧都比较小
        dfs(cur.left);
        // 如果递归左侧之后pre为null，说明当前cur是第一个节点
        if (pre == null) {
            head = cur;
        } else {
            // 否则让pre指向cur
            pre.right = cur;
        }
        cur.left = pre;
        // 在递归右子树前，让cur变成pre
        // 完成后，pre指向最后一个节点
        pre = cur;
        dfs(cur.right);
    }
}
```
### 37. 序列化二叉树
```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Codec {

    // Encodes a tree to a single string.
    public String serialize(TreeNode root) {
        if(root == null) return "[]";
        StringBuilder res = new StringBuilder("[");
        Queue<TreeNode> queue = new LinkedList<>() {{ add(root); }};
        while(!queue.isEmpty()) {
            TreeNode node = queue.poll();
            if(node != null) {
                res.append(node.val + ",");
                queue.add(node.left);
                queue.add(node.right);
            }
            else res.append("null,");
        }
        res.deleteCharAt(res.length() - 1);
        res.append("]");
        return res.toString();

    }

    // Decodes your encoded data to tree.
    public TreeNode deserialize(String data) {
        if(data.equals("[]")) return null;
        String[] vals = data.substring(1, data.length() - 1).split(",");
        TreeNode root = new TreeNode(Integer.parseInt(vals[0]));
        Queue<TreeNode> queue = new LinkedList<>() {{ add(root); }};
        int i = 1;
        while(!queue.isEmpty()) {
            TreeNode node = queue.poll();
            if(!vals[i].equals("null")) {
                node.left = new TreeNode(Integer.parseInt(vals[i]));
                queue.add(node.left);
            }
            i++;
            if(!vals[i].equals("null")) {
                node.right = new TreeNode(Integer.parseInt(vals[i]));
                queue.add(node.right);
            }
            i++;
        }
        return root;
    }
}

// Your Codec object will be instantiated and called as such:
// Codec codec = new Codec();
// codec.deserialize(codec.serialize(root));
```



### 38. 字符串的排列
dfs，先排序，以便去除重复字符串。去除方法是，在遍历DFS时，如果当前字符和前一个字符一样，且前一个字符在当前dfs层级才被第一次使用，那么当前字符就跳过。  
```
class Solution {
    
    List<String> res;
    char[] ch;
    int len;
    public String[] permutation(String s) {
        res = new ArrayList<>();
        ch = s.trim().toCharArray();
        Arrays.sort(ch);
        len = ch.length;
        boolean[] used = new boolean[len];

        dfs(0, used, "");
        return res.toArray(new String[res.size()]);
    }

    private void dfs(int cur, boolean[] used, String s) {
        // System.out.println(s);
        if (cur == len) {
            res.add(s);
            return;
        }

        for (int i = 0; i < len; i++) {
            // 注意处理不能有重复答案
            if (used[i] || (i > 0 && ch[i] == ch[i - 1] && !used[i - 1])) {
                continue;
            }
            used[i] = true;
            dfs(cur + 1, used, s + ch[i]);
            used[i] = false;
        }
    }
}
```
### 39. 数组中出现次数超过一半的数字
答案数字数量超过一半，那么将数组排序后，一半位置的数字一定是答案
```
class Solution {
    public int majorityElement(int[] nums) {
        Arrays.sort(nums);
        return nums[nums.length / 2];
    }
}
```
### 40. 最小的k个数
数据量小的话直接排序速度最快，如果数据量特别大，就用堆  
```
class Solution {
    public int[] getLeastNumbers(int[] arr, int k) {
        int[] res = new int[k];
        Arrays.sort(arr);
        for (int i = 0; i < k; i++) {
            res[i] = arr[i];
        }
        return res;
    }
}
```

### 41. 数据流中的中位数
一个大顶堆，一个小顶堆。维持两个的大小总是相等或者差1.  这里我们维护小顶堆大一点。  
当来了新num，如果堆大小相等，则应该插入小顶堆，插入方法是，先插入大顶堆，这时大顶堆溢出的元素插入小顶堆；
如果小顶堆比较大，则直接插入大顶堆。  
最终的中位数结果，比较两个堆的大小，如果大小相等，则中位数是两个堆顶元素的平均；否则是小顶堆的堆顶。  
```
class MedianFinder {
    PriorityQueue<Integer> minQueue;
    PriorityQueue<Integer> maxQueue;
    /** initialize your data structure here. */
    public MedianFinder() {
        // 小顶堆，保存大的一半
        minQueue = new PriorityQueue<>();
        // 大顶堆，保存小的一半
        maxQueue = new PriorityQueue<>((x, y) -> y - x);
    }
    
    public void addNum(int num) {
        if (minQueue.size() <= maxQueue.size()) {
            maxQueue.offer(num);
            minQueue.offer(maxQueue.poll());
        } else {
            minQueue.offer(num);
            maxQueue.offer(minQueue.poll());
        }
    }
    
    public double findMedian() {
        return minQueue.size() == maxQueue.size() ? (minQueue.peek() + maxQueue.peek()) / 2.0 : minQueue.peek();
    }
}

/**
 * Your MedianFinder object will be instantiated and called as such:
 * MedianFinder obj = new MedianFinder();
 * obj.addNum(num);
 * double param_2 = obj.findMedian();
 */
```
### 42. 连续子数组的最大和
动态规划，dp[i]记录当前数字为结尾时，连续子数组的最大和。  
```
class Solution {
    public int maxSubArray(int[] nums) {
        int len = nums.length;
        int[] dp = new int[len];

        dp[0] = nums[0];
        int max = dp[0];
        for (int i = 1; i < len; i++) {
            int tmp = dp[i - 1] + nums[i];
            
            if (tmp < nums[i]) {
                dp[i] = nums[i];
            } else {
                dp[i] = tmp;
            }
            max = Math.max(max, dp[i]);
        }
        return max;
    }
}
```

### 43. 1～n 整数中 1 出现的次数
所有1出现的次数 = 个位1出现次数 + 十位1出现次数 + 百位1出现次数 + 。。。 + 最高位1出现次数  
将数字按照当前统计的位，分为high，cur，low  
当前位1出现次数有三种情况：  
1. 当前位是0：1出现次数为high * digit
2. 当前位是1：1出现次数为high * digit + low + 1
3. 当前位是其他：1出现次数为（high + 1） * digit
累加即可  
```
class Solution {
    public int countDigitOne(int n) {
        int low = 0, high = n / 10;
        int cur = n % 10;
        int digit = 1;
        int result = 0;

        while (high != 0 || cur != 0) {
            if (cur == 0) {
                result += high * digit;
            } else if (cur == 1) {
                result += high * digit + low + 1;
            } else {
                result += (high + 1) * digit;
            }
            digit *= 10;
            cur = high % 10;
            high = high / 10;
            low = n % digit;

        } 
        return result;
    }
}
```

### 44.数字序列中某一位的数字
先统计在多少位的数字范围中出现，然后统计在这个位数的哪一个数字中出现，最后统计在这个数字的第几位。  

```
class Solution {
    public int findNthDigit(int n) {
        int digit = 1;
        long start = 1;
        long count = 9;

        while (n > count) {
            n -= count;
            digit += 1;
            start *= 10;
            count = 9 * start * digit;
        }
		// 是从start开始的第(n - 1) / digit个数字，大小就是num
        long num = start + (n - 1) / digit;
		// num的第(n - 1) % digit位
        return String.valueOf(num).charAt((n - 1) % digit) - '0';
    }
}
```

### 45. 把数组排成最小的数
字符串排序，自定义比较器，排序规则是，前拼接后，小于后拼接前  
```
class Solution {
    public String minNumber(int[] nums) {
        List<String> list = new ArrayList<>();
        for (int i : nums) {
            list.add(String.valueOf(i));
        }
        list.sort((o1, o2) -> (o1 + o2).compareTo(o2 + o1));
        return String.join("", list);
    }
}
```

### 46.把数字翻译成字符串
跟青蛙跳台阶类似。  
只有当前两个数字是[10, 25]之间时，才能够合在一起翻译，注意'06'这样的情况是不能作为一体翻译的  
```
class Solution {
    public int translateNum(int num) {
        if (num < 10) {
            return 1;
        }
        char[] nums = String.valueOf(num).toCharArray();
        int len = nums.length;
        int[] dp = new int[len];

        dp[0] = 1;
        if (nums[0] == '1' || (nums[0] == '2' && nums[1] <= '5')) {
            dp[1] = 2;
        } else {
            dp[1] = 1;
        }

        for (int i = 2; i < len; i++) {
            if (nums[i - 1] == '1' || (nums[i - 1] == '2' && nums[i] <= '5')) {
                dp[i] = dp[i - 1] + dp[i - 2];
            } else {
                dp[i] = dp[i - 1];
            }
            
        }

        return dp[len - 1];
    }   
}
```

### 47. 礼物的最大价值
动态规划，二维数组记录一下即可  
```
class Solution {
    public int maxValue(int[][] grid) {
        if (grid.length == 0 || grid[0].length == 0) {
            return 0;
        }   
        int m = grid.length;
        int n = grid[0].length;

        int[][] dp = new int[m][n];
        dp[0][0] = grid[0][0];

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (i == 0 && j == 0) continue;
                if (i == 0) {
                    // 初始化第一行
                    dp[i][j] = dp[i][j - 1] + grid[i][j]; 
                } else if (j == 0) {
                    // 初始化第一列
                    dp[i][j] = dp[i - 1][j] + grid[i][j]; 
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j] + grid[i][j], dp[i][j - 1] + grid[i][j]);
                }
            }
        }
        return dp[m - 1][n - 1];
    }
}
```

### 48. 最长不含重复字符的子字符串
滑动窗口。 开一个Set记录窗口中已经出现的字符，当窗口中碰到重复字符，则从窗口左边开始移除字符，直到把重复字符移除掉，即收缩窗口的过程；没有碰到重复字符则窗口向右扩展，直到最后。  
```
class Solution {
    public int lengthOfLongestSubstring(String s) {
        if (s.length() == 0) return 0;
        int res = 0;
        // 窗口左
        int start = 0;
        // 窗口右
        int i = 0;
        Set<Character> set = new HashSet<>();
        
        while (i < s.length()) {
            char cur = s.charAt(i);
            while (set.contains(cur)) {
                set.remove(s.charAt(start++));
            }

            set.add(cur);
            res = Math.max(res, i - start + 1);
            i++;
        }

        return res;
    }
}
```

### 49. 丑数
既然要找只有2，3，5为因数的数，那么可以用前面的数，乘以2，3，5，看哪个数更小，小的那个作为本次添加的对象。2，3，5分别记录前一次乘过的数字。  
```
class Solution {
    public int nthUglyNumber(int n) {
        int[] dp = new int[n];
        dp[0] = 1;
        int p2 = 0, p3 = 0, p5 = 0;
        for (int i = 1; i < n; i++) {
            int tmp = Math.min(Math.min(dp[p2] * 2, dp[p3] * 3), dp[p5] * 5);
            dp[i] = tmp;
            if (tmp == dp[p2] * 2) {
                p2++;
            }
            if (tmp == dp[p3] * 3) {
                p3++;
            }
            if (tmp ==  dp[p5] * 5) {
                p5++;
            }
        }
        return dp[n - 1];
    }
}
```

### 50. 第一个只出现一次的字符
利用HashMap即可，可以用char数组作为字符出现的顺序，也可以利用LinkedHashMap的顺序性记录顺序。  
```
class Solution {
    public char firstUniqChar(String s) {
        Map<Character, Integer> map = new LinkedHashMap<>();

        for (char i : s.toCharArray()) {
            map.put(i, map.getOrDefault(i, 0) + 1);
        }

        for (char c : map.keySet()) {
            if (map.get(c) == 1) {
                return c;
            }
        }
        return ' ';
    }   
}
```

### 51. 数组中的逆序对
暴力法超时。  
逆序数可以用归并排序分治来做，方法是:
* 分治左右数组
* 在合并时，如果出现了左边数组的数字放在右边，右边数组的数字放在左边的情况，那么增加一次res，增加的数量是**左边子数组剩余还没有合并的数字的数量**，因为左边剩余的数字都比右边当前添加的那个数字大。  
```
class Solution {
    // 全局变量记录结果
    int res = 0;

    public int reversePairs(int[] nums) {
        if (nums.length < 2) return 0;

        sort(nums);
        return res;
    }

    private void sort(int[] nums) {
        int[] tmp = new int[nums.length];
        sort(nums, tmp, 0, nums.length - 1);
    }

    private void sort(int[] nums, int[] tmp, int left, int right) {
        if (left < right) {
            int mid = left + (right - left) / 2;
            sort(nums, tmp, left, mid);
            sort(nums, tmp, mid + 1, right);
            merge(nums, tmp, left, mid, right);
        }
    }

    private void merge(int[] nums, int[] tmp, int left, int mid, int right) {
        int i = left;
        int j = mid + 1;
        int t = 0;
        while (i <= mid && j <= right) {
            if (nums[i] <= nums[j]) {
                tmp[t++] = nums[i++];
            } else {
                tmp[t++] = nums[j++];
                // 这里统计逆序数，逆序数的个数是，左边子数组剩余没有合并的数字的数量  
                res += mid - i + 1;
            }
        }

        while (i <= mid) {
            tmp[t++] = nums[i++];
        }

        while (j <= right) {
            tmp[t++] = nums[j++];
        }

        t = 0;
        while (left <= right) {
            nums[left++] = tmp[t++];
        }
    }
}
```

### 52. 两个链表的第一个公共节点
方法一：set记录，很慢
```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        Set<ListNode> set = new HashSet<>();

        ListNode ptr = headA;
        while (ptr != null) {
            set.add(ptr);
            ptr = ptr.next;
        }
        
        ptr = headB;
        while (ptr != null) {
            if (set.contains(ptr)) {
                return ptr;
            }
            ptr = ptr.next;
        }
        return null;
    }
}
```

**方法二：双指针找交叉节点**
```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        ListNode ptr1 = headA;
        ListNode ptr2 = headB;
        ListNode result = null;
        while (ptr1 != null && ptr2 != null) {
            if (ptr1 == ptr2) {
                return ptr1;
            }

            ptr1 = ptr1.next;
            ptr2 = ptr2.next;
        }

        while (ptr1 != null) {
            ptr1 = ptr1.next;
            headA = headA.next;
        }

        while (ptr2 != null) {
            ptr2 = ptr2.next;
            headB = headB.next;
        }

        while (headA != null && headB != null) {
            if (headA == headB) {
                return headA;
            }

            headA = headA.next;
            headB = headB.next;
        }
        return null;
    }
}
```

### 53 - I. 在排序数组中查找数字 I
二分查找。找到目标数字之后，往两边扩展计数即可。  
```
class Solution {
    public int search(int[] nums, int target) {

        // Arrays.sort(nums);
        int res = 0;
        int left = 0;
        int right = nums.length - 1;
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] < target) {
                left = mid + 1;
            } else if (nums[mid] > target) {
                right = mid - 1;
            } else {
                int ptr = mid;
                while (ptr >= left) {
                    if (nums[ptr--] == target) {
                        res++;
                    }
                }

                ptr = mid + 1;
                while (ptr <= right) {
                    if (nums[ptr++] == target) {
                        res++;
                    }
                }
                break;
            }
        } 
        return res;
    }
}
```

### 53 - II. 0～n-1中缺失的数字
二分查找。  
将数组分为两部分，没有缺失的左子数组和有缺失的右子数组，mid查找右子数组的首位元素。
i <= j作为循环条件，循环跳出前一步，一定是i == j == m，此时三者应当都指向的是右子数组的首位。当循环跳出时，i > j，那么此时i指向右子数组首位，即为结果  


```
class Solution {
    public int missingNumber(int[] nums) {
        int left = 0;
        int right = nums.length - 1;
        int mid = 0;
        while (left <= right) {
            mid = left + (right - left) / 2;
            if (nums[mid] == mid) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        
        return left;
    }
}
```

### 54. 二叉搜索树的第k大节点
二叉搜索树中，中序遍历的结果为递增结果；则中序遍历的倒序为递减结果。  
中序遍历是左，中，右的顺序，那逆序是右，中，左。递归方法实现  
```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    int k;
    int res;

    public int kthLargest(TreeNode root, int k) {
        this.k = k;
        res = 0;
        dfs(root);
        return res;
    }

    private void dfs(TreeNode node) {
        if (node == null || k == 0) return;
        dfs(node.right);
        if (--k == 0) {
            res = node.val;
            return;
        }
        dfs(node.left);
    }
}
```

### 55 - I. 二叉树的深度
深度优先搜索即可  

```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public int maxDepth(TreeNode root) {
        return dfs(root, 1);
    }

    private int dfs(TreeNode node, int depth) {
        if (node == null) {
            return depth - 1;
        }

        if (node.left == null && node.right == null) {
            return depth;
        }

        return Math.max(dfs(node.left, depth + 1), dfs(node.right, depth + 1));
    }
}
```

### 55 - II. 平衡二叉树
深度优先搜索。让-1作为左右不平衡的标志，如果遍历过程中出现-1，则说明下面已经有子树不平衡了，其他的遍历过程直接返回-1即可。  

```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public boolean isBalanced(TreeNode root) {
        if (root == null) {
            return true;
        }
        return dfs(root, 1) != -1;
    }

    private int dfs(TreeNode node, int depth) {
        if (node == null) {
            return depth - 1;
        }

        // if (node.left == null && node.right == null) return depth;

        int left = dfs(node.left, depth + 1);
        int right = dfs(node.right, depth + 1);

        return left != -1 && right != -1 && Math.abs(left - right) <= 1 ? Math.max(left, right) : -1;
    }
}

```

### 56 - I. 数组中数字出现的次数
方法一：排序后比较，速度较慢
```
class Solution {
    public int[] singleNumbers(int[] nums) {
        Arrays.sort(nums);
        int[] res = new int[2];
        int k = 0;
        if (nums[0] != nums[1]) res[k++] = nums[0];
        
        for (int i = 1; i < nums.length - 1; i++) {
            if (k == 2) return res;
            if (nums[i] != nums[i - 1] && nums[i] != nums[i + 1]) {
                res[k++] = nums[i];
            }
        }

        if (k < 2) res[k] = nums[nums.length - 1];
        return res;
    }
}
```

**方法二：位运算**，直接看注释   

```
class Solution {
    public int[] singleNumbers(int[] nums) {
        int sum = nums[0];
        for (int i = 1; i < nums.length; i++) {
            sum ^= nums[i];
        }

        int mask = 1;
        // 因为有两个不同的数字，所以肯定至少有一位异或的结果是1
        while ((sum & mask) == 0) mask <<= 1;

        int a = 0;
        int b = 0;

        for (int num : nums) {
            // 根据num的mask位是0还是不等于0分为两组，相同的数字一定会被分在同一组，两组分别异或在一起，得到的肯定是结果
            if ((num & mask) == 0) {
                a ^= num;        
            } else {
                b ^= num;
            }
        }
        return new int[]{a, b};
    }
}
```

### 56 - II. 数组中数字出现的次数 II
位运算，既然每个数字出现三次，那么统计每个位上1出现的次数，如果出现4次，那么结果的这个位必然是1。  

```
class Solution {
    public int singleNumber(int[] nums) {
        int[] count = new int[32];

        for (int num : nums) {
            for (int i = 0; i < 32; i++) {
                count[i] += (num & 1);
                num >>>= 1;
            }
        }

        int res = 0;
        for (int i = 31; i >= 0; i--) {
            // 先左移，因为要先空出符号位
            res <<= 1;
            // 这里用+= 也可以，但是|=更快
            res |= count[i] % 3;
        }
        return res;
    }
}
```

### 57. 和为s的两个数字
双指针，从两头往中间走，大了就收缩右边，小了就收缩左边  

```
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;

        while (left < right) {
            int sum = nums[left] + nums[right];
            if (sum == target) return new int[]{nums[left], nums[right]};
            else if (sum < target) {
                left++;
            } else {
                right--;
            }
        }
        return new int[]{-1, -1};
    }
}
```

### 57 - II. 和为s的连续正数序列
滑动窗口。记录滑动窗口的和，如果小了就向右扩展，大了就收缩左边  
```
class Solution {
    public int[][] findContinuousSequence(int target) {
        int left = 1;
        int right = 1;
        int sum = 1;

        List<int[]> list = new ArrayList<>();
        int maxLen = 0;
        
        while (right < target) {
            if (sum == target) {
                int[] tmp = new int[right - left + 1];
                for (int i = left; i <= right; i++) {
                    tmp[i - left] = i;
                }
                list.add(tmp);
                maxLen = Math.max(maxLen, tmp.length);
                sum -= left;
                left++;
            }
            else if (sum < target) {
                right++;
                sum += right;
            } else {
                sum -= left;
                left++;
            }
        }
        // 注意这里List转数组的API
        return list.toArray(new int[list.size()][]);
    }
}
```

### 58 - I. 翻转单词顺序
先用空格分割字符串，分割出的结果不是空格或者空的就拼接  
```
class Solution {
    public String reverseWords(String s) {
        String[] ss = s.trim().split(" ");
        StringBuilder sb = new StringBuilder();

        for (int i = ss.length - 1; i >= 0; i--) {
            if (ss[i].equals(" ") || ss[i].equals("")) continue;
            else {
                sb.append(ss[i]);
                if (i != 0) sb.append(" ");
            }
            // System.out.println(sb.toString());
        }
        return sb.toString();
    }
}
```

### 58 - II. 左旋转字符串
直接把左半部分截下来拼上即可  

```
class Solution {
    public String reverseLeftWords(String s, int n) {
        if (n > s.length()) {
            return s;
        } else if (n == s.length()) {
            return s;
        } 

        String front = s.substring(0, n);
        return s.substring(n, s.length()) + front;
    }
}
```

### 59 - I. 滑动窗口的最大值
**双端队列的典型题。**维持一个双端队列，里面存储当前i为开头时，窗口中的最大值递减序列。  
当最大值滑出窗口，则删除队头元素；当新元素滑入窗口时，将其添加进双端队列，并删除队尾所有小于当前新元素的值，以维护双端队列递减。  
这里双端队列递减的意义是，每次窗口移动，滑出一个元素，而窗口最大值一直在队列中。  

```
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        if (nums.length == 0 || k == 0) return new int[0];
        Deque<Integer> deque = new ArrayDeque<>();
        int[] res = new int[nums.length - k + 1];
        // 未形成窗口
        for(int i = 0; i < k; i++) {
            while(!deque.isEmpty() && deque.peekLast() < nums[i])
                deque.removeLast();
            deque.addLast(nums[i]);
        }
        res[0] = deque.peek();
        // 形成窗口后
        for (int i = 1; i < nums.length - k + 1; i++) {
            // System.out.println(deque.toString());
            if (nums[i - 1] == deque.peekFirst()) {
                deque.removeFirst();
            }

            while (!deque.isEmpty() && deque.getLast() < nums[i + k - 1]) {
                deque.removeLast();
            }
            deque.addLast(nums[i + k - 1]);
            res[i] = deque.peekFirst();
        }
        return res;
    }
}
```

### 59 - II. 队列的最大值
跟上一题一样，双端队列维护最大值  

```
class MaxQueue {
    
    Deque<Integer> maxDeque;
    Queue<Integer> queue;
    
    public MaxQueue() {
        maxDeque = new ArrayDeque<>();
        queue = new ArrayDeque<>();
    }
    
    public int max_value() {
        if (queue.isEmpty()) return -1;
        return maxDeque.peekFirst();
    }
    
    public void push_back(int value) {
        queue.offer(value);
        while (!maxDeque.isEmpty() && maxDeque.getLast() < value) {
            maxDeque.removeLast();
        }
        maxDeque.offer(value);
    }
    
    public int pop_front() {
        if (queue.isEmpty()) return -1;
        int value = queue.poll();
        if (maxDeque.peek() == value) {
            maxDeque.poll();
        }
        return value;
    }   
}

/**
 * Your MaxQueue object will be instantiated and called as such:
 * MaxQueue obj = new MaxQueue();
 * int param_1 = obj.max_value();
 * obj.push_back(value);
 * int param_3 = obj.pop_front();
 */
```

### 60. n个骰子的点数
动态规划。  投n颗色子，所有可能的点数情况是6 * n - (n - 1)，所有可能投出的每颗色子点数组合数量是6^n，也就是每种点数情况对应多种组合。  

当前投第n颗色子时，每个点数被投出的组合数，是n-1颗色子投递出的点数演变来的。  

比如当前第二颗色子投出的10点，可能是第一颗色子4 + 第二颗色子6，5 + 5， 6 + 4。 那么想到动态规划。  

定义dp状态：第一维记录当前色子投递的颗数，第二维记录色子投每种点数的组合数。  

状态转移：  
```
for (第n枚骰子的点数 i = 1; i <= 6; i ++) {
    dp[n][j] += dp[n-1][j - i]
}
```
初始化：第一次投递色子1~6每种组合数都是1.  

```
class Solution {
    public double[] dicesProbability(int n) {
        // 第一维是扔了多少枚色子
        // 第二维是扔中每一点的组合数
        int[][] dp = new int[n + 1][6 * n + 1];

        for (int i = 1; i <= 6; i++) {
            dp[1][i] = 1;
        }

        // i控制扔第几枚色子
        for (int i = 2; i <= n; i++) {
            // j控制当前投递色子的最终累加点数
            for (int j = i; j <= 6 * i; j++) {
                // cur控制与上一枚色子投递完的状态之间的转换
                for (int cur = 1; cur <= 6; cur++) {
                    if (j - cur <= 0) continue; 
                    dp[i][j] += dp[i - 1][j - cur];
                }
            }
        }
        
        double[] res = new double[5 * n + 1];
        int sum = 0;
        for (int i = n; i <= 6 * n; i++) {
            sum += dp[n][i];
        }

        // System.out.println(Arrays.deepToString(dp));
        for (int i = n; i <= 6 * n; i++) {
            res[i - n] = dp[n][i] / (double)sum;
        }
        return res;
    }
}
```

由于dp每一行只由上一行演变来，所以优化空间，只定义两个dp数组，交替演变.此时空间复杂度由原本的n^2转为O(n)  

```
class Solution {
    public double[] dicesProbability(int n) {
        // 第一维是扔了多少枚色子
        // 第二维是扔中每一点的组合数
        int[] dp = new int[7];

        for (int i = 1; i <= 6; i++) {
            dp[i] = 1;
        }

        // i控制扔第几枚色子
        for (int i = 2; i <= n; i++) {
            int[] tmp = new int[i * 6 + 1];
            // j控制当前投递色子的最终累加点数
            for (int j = i; j <= 6 * i; j++) {
                // cur控制与上一枚色子投递完的状态之间的转换
                for (int cur = 1; cur <= 6; cur++) {
                    if (j - cur <= 0 || j - cur >= dp.length) continue; 
                    tmp[j] += dp[j - cur];
                }
            }
            dp = tmp;
        }
        
        double[] res = new double[5 * n + 1];
        int sum = 0;
        for (int i = n; i <= 6 * n; i++) {
            sum += dp[i];
        }

        // System.out.println(Arrays.deepToString(dp));
        for (int i = n; i <= 6 * n; i++) {
            res[i - n] = dp[i] / (double)sum;
        }
        return res;
    }
}
```

### 61. 扑克牌中的顺子
先排序，然后记录有几个0，可以替代几个任意牌。  

然后顺序遍历，前后两个比较，有几种不满足顺子的情况：
1. 前后两张牌相等
2. 前后两张牌差距过大，大于了替代牌的数量

```
class Solution {
    public boolean isStraight(int[] nums) {
        Arrays.sort(nums);
        int zeroCount = 0;
        
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] == 0) {
                zeroCount++;
                continue;
            }

            if (i < nums.length - 1) {
                if (nums[i] == nums[i + 1]) return false;
                if (nums[i] != nums[i + 1] - 1 && zeroCount < nums[i + 1] - nums[i] - 1) return false;
            }
        }
        return true;
    }
}
```

### 62. 圆圈中最后剩下的数字
方法一：用链表模拟，超时  

```
class Solution {
    public int lastRemaining(int n, int m) {
        List<Integer> list = new LinkedList<>();
        int ptr = 0;

        for (int i = 0; i < n; i++) {
            list.add(i);
        }

        while (n > 1) {
            ptr = (ptr + m - 1) % n;
            list.remove(ptr);
            n--;
        }
        return list.get(0);
    }
}
```

**方法二：约瑟夫环，数学解法**  

我们把每一轮删掉一个数字看作把删掉数字之后的数字挪到数组最前面。最后剩下的一个数字下标是0，我们可以反推，经过反推，它上一轮的位置是`(当前index + m) % 上一轮剩余数字的个数`，一直反推到数组长度为n  

```
class Solution {
    public int lastRemaining(int n, int m) {
        int res = 0;
        for (int i = 2; i <= n; i++) {
            res = (res + m) % i;
        }
        return res;
    }
}
```

### 63.股票的最大利润
动态规划，dp数组记录当天卖出的最大收益。  

最大收益 = 当前位置股票的价格 - 前面最低的买入价格  

所以设置一个变量cheap记录买入的最低价格  

```
class Solution {
    public int maxProfit(int[] prices) {
        if (prices.length == 0) return 0;
        int len = prices.length;
        
        int[] dp = new int[len];
        dp[0] = 0;
        int cheap = 0;
        int res = 0;
        for (int i = 1; i < len; i++) {
            if (prices[i] < prices[cheap]) {
                cheap = i;
            }

            dp[i] = prices[i] - prices[cheap];

            res = Math.max(res, dp[i]);
        }
        return res;
    }
}
```

### 64. 求1+2+…+n 
普通递归，必须要用if判断递归结束  

```
public int sumNums(int n) {
    if(n == 1) return 1;
    n += sumNums(n - 1);
    return n;
}
```

可以用**双目运算符的短路效应来判断递归，避免使用if**  
这里使用A&&B，如果A为false，那么B不会执行，如果A为true，那么B会执行。用n>1作为A，sumNums(n - 1)作为B，则当n<=1的时候，就不再递归执行B。  

```
class Solution {
    int res = 0;
    public int sumNums(int n) {
        boolean x = n > 1 && sumNums(n - 1) > 0;
        res += n;
        return res;
    }
}

```

### 65. 不用加减乘除做加法
最清晰的一个题解  

https://leetcode-cn.com/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/solution/jin-zhi-tao-wa-ru-he-yong-wei-yun-suan-wan-cheng-j/  

```
class Solution {
    public int add(int a, int b) {
        // 当还有进位的时候就一直循环
        // 让非进位和 和 进位 相加的过程，与a + b是一样的过程，所以循环计算
        while (b != 0) {
            // 不考虑进位的和
            int sum = a ^ b;
            // 单独算进位
            int carry = (a & b) << 1;
            a = sum;
            b = carry;
        }
        return a;
    }
}
```
### 66. 构建乘积数组
开两个数组，一个记录左半边的成绩乘积，一个记录右半边的乘积。  
```
class Solution {
    public int[] constructArr(int[] a) {
        int len = a.length;
        if (len == 0) return new int[0];
        int[] res = new int[len];
        int[] left = new int[len];
        int[] right = new int[len];
        left[0] = 1;
        right[len - 1] = 1;
        for (int i = 1; i < len; i++) {
            left[i] = left[i - 1] * a[i - 1];
            right[len - 1 - i] = right[len - i] * a[len - i];
        }

        for (int i = 0; i < len; i++) {
            if (i == 0) res[i] = right[i];
            else if (i == len - 1) res[i] = left[i];
            res[i] = left[i] * right[i];
        }
        return res;
    }
}
```

### 67. 把字符串转换成整数
变成字符数组一位一位的加即可，注意超限的处理。  

```
class Solution {
    public int strToInt(String str) {
        char[] chars = str.trim().toCharArray();
        int flag = 1;
        int res = 0;
        int bndry = Integer.MAX_VALUE / 10;
        for (int i = 0; i < chars.length; i++) {
            
            if (i == 0 && chars[i] == '-') {
                flag = -1;
                continue;
            }
            if (i == 0 && chars[i] == '+') {
                flag = 1;
                continue;
            }

            if (Character.isDigit(chars[i])) {
                // 如果要超限了
                if (res > bndry || (res == bndry && chars[i] > '7')) {
                    return flag == 1 ? Integer.MAX_VALUE : Integer.MIN_VALUE;
                }
                res = res * 10 + (chars[i] - '0');
            } else {
                if (res > 0) break;
                else return 0;
            }
        }
        return res * flag;
    }
}
```

### 68 - I. 二叉搜索树的最近公共祖先
递归搜索，利用二叉搜索树的性质，如果搜索到的节点等于p或者等于q，那么node是公共祖先。  

如果搜索到的节点值，介于p的值和q的值之间，那么node是公共祖先，否则，根据p、q的值决定左子树还是右子树。  

```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    TreeNode res;
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        dfs(root, p, q);
        return res;
    }

    private void dfs(TreeNode node, TreeNode p, TreeNode q) {
        if (node == null) return;
        if (node == p || node == q) {
            res = node;
            return;
        }

        if ((node.val > p.val && node.val < q.val) || (node.val < p.val && node.val > q.val)) {
            res = node;
            return;
        }
        if (node.val < p.val) {
            dfs(node.right, p, q);
        } else {
            dfs(node.left, p, q);
        }
    }
}
```

### 68 - II. 二叉树的最近公共祖先
从上往下递归遍历，三种情况：
1. root是p或者q，则root就是最近公共节点
2. p和q都在root的单侧子树上，则继续往下递归这一侧子树
3. p和q分属root的两侧子树，则root就是最近公共节点


```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null || root == p || root == q) return root;
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);
        if (left == null) return right;
        if (right == null) return left;
        return root;
    }
}
```