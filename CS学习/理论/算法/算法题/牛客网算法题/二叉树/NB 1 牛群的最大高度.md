就不采用, 递归的方式来做了, 自己弄个栈来做
用栈来保存路径, curr 表示当前的节点, pre 保留往回走时的上一步

如果是 用递归来做 它的栈链路是这样的, 可以做下参考
![[Pasted image 20240508205528.png|left|500]]
黄色表示返回

用栈模拟的话, 不可能模拟得一摸一样, 递归的话一个栈会经过3次, 第三次后就不会再碰到这个栈了, 可以和它拜拜了, 
换做模拟来说, 只要是第3次使用该栈节点, 就可以把这个节点给弹出了


curr 的 遍历 策略
一路向左走, 触底了, curr == null 了, 就要往回走了, 就看栈顶节点的右边有没有, 右边有且触底不是从右边上来的, 就往右走

往右走后还是一样的策略, 触底一路返回, 拿到栈顶元素, 发现是从右边上来的(即第栈顶节点被第三次使用), 把栈顶节点弹出, 继续返回, 直到栈为空为止

把怎么遍历二叉树弄明白后, 咱们只需要简简单单在前序遍历的位置完成最大值的测试及更新即可

```java
public class Solution {
    public int findMaxHeight (TreeNode root) {
        int max = 0;
        TreeNode pre = null;
        TreeNode curr = root;
        Stack<TreeNode> stack = new Stack<>();
        while(curr != null || !stack.isEmpty()) {
            if(curr != null) {
                stack.push(curr); // 第一次使用
                max = Math.max(max, curr.val); 
                curr = curr.left;
            } else {
                TreeNode parent = stack.peek();
                if(parent.right == null || pre == parent.right) { // 第三次使用
                    pre = stack.pop();// 保留返回时的上一步的记录, 用来做第几次使用的测试
                } else { // 第二次使用
                    curr = parent.right;
                }
            }
        }
        return max;
    }
}
```
