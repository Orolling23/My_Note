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
