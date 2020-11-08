### 【题目】[*剑指 Offer 32 - III. 从上到下打印二叉树 III](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/)
请实现一个函数按照之字形顺序打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行以此类推。

例如:

	给定二叉树: [3,9,20,null,null,15,7],
	    3
	   / \
	  9  20
	    /  \
	   15   7
	返回其层次遍历结果：
	[
	  [3],
	  [20,9],
	  [15,7]
	]

提示：
节点总数 <= 1000

### 【解题思路1】头插法或者翻转
```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> ans = new ArrayList<>();
        if(root == null)    return ans;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while(!queue.isEmpty()) {
            int size = queue.size();
            LinkedList<Integer> cur = new LinkedList<>();
            for(int i = 0; i < size; i++) {
                TreeNode temp = queue.poll();
                if(ans.size() % 2 == 0) {
                    cur.add(temp.val);
                } else {
                    cur.addFirst(temp.val);
                }
                if(temp.left != null)   queue.offer(temp.left);
                if(temp.right != null)  queue.offer(temp.right);
            }
            // 或者在这里翻转if(ans.size() % 2 == 1) Collections.reverse(tmp);
            ans.add(cur);
        }
        return ans;
    }
}
```

**时间复杂度**：O(N)。N 为二叉树的节点数量，即 BFS 需循环 N 次，占用 O(N) ；双端队列的队首和队尾的添加和删除操作的时间复杂度均为 O(1) 。
**空间复杂度**：O(N)。最差情况下，即当树为满二叉树时，最多有 N/2 个树节点 同时 在 deque 中，使用 O(N) 大小的额外空间。

### 【解题思路2】倒序入队

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        Deque<TreeNode> deque = new LinkedList<>();
        List<List<Integer>> res = new ArrayList<>();
        if(root != null) deque.add(root);
        while(!deque.isEmpty()) {
            // 打印奇数层
            List<Integer> tmp = new ArrayList<>();
            int size = deque.size();
            for(int i = 0; i < size; i++) {
                // 从左向右打印
                TreeNode node = deque.removeFirst();
                tmp.add(node.val);
                // 先左后右加入下层节点
                if(node.left != null) deque.addLast(node.left);
                if(node.right != null) deque.addLast(node.right);
            }
            res.add(tmp);
            if(deque.isEmpty()) break; // 若为空则提前跳出
            // 打印偶数层
            tmp = new ArrayList<>();
            size = deque.size();
            for(int i = 0; i < size; i++) {
                // 从右向左打印
                TreeNode node = deque.removeLast();
                tmp.add(node.val);
                // 先右后左加入下层节点
                if(node.right != null) deque.addFirst(node.right);
                if(node.left != null) deque.addFirst(node.left);
            }
            res.add(tmp);
        }
        return res;
    }
}
```

