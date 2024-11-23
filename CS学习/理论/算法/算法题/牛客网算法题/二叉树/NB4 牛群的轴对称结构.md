[原题链接]([牛群的轴对称结构_牛客题霸_牛客网 (nowcoder.com)](https://www.nowcoder.com/practice/a200535760fb4da3a4568c03c1563689?tpId=354&tqId=10591485&ru=/exam/oj&qru=/ta/interview-202-top/question-ranking&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3D%25E7%25AE%2597%25E6%25B3%2595%25E7%25AF%2587%26topicId%3D354))

这道题很适合用层序遍历来做, 我们需要判断每一层是否为回文结构
即 要每一层节点的值连起来是否为回文 和 每一层的节点的子树结构是否对称
我们可以用栈验证
因为遍历每一层时, 我们可以前半段遍历时, 往栈中放元素, 在后半段弹出元素判断是否与当前遍历的节点的值相等来判断是否为回文
子树的结构同理
最后, 再看看栈是否为空, 不为空说明不对称;


```java
public class Solution {  
    public boolean isSymmetric (TreeNode root) {  
        Queue<TreeNode> queue = new LinkedList<>();  
        queue.add(root);  
        while(!queue.isEmpty()) {  
            Stack<Integer> valStack = new Stack<>();  
  
            Stack<Integer> nodeStack = new Stack<>();  
  
            int size = queue.size();  
  
            for(int i = 1; i <= size; i++) {  
                TreeNode poll =  queue.poll();  
  
                if(i <= size / 2) valStack.push(poll.val);  
                else if(i > (size + 1) / 2) if(poll.val != valStack.pop()) return false;  
  
                if(poll.left != null) {  
                    queue.add(poll.left);  
                    if(i <= (size + 1) / 2) nodeStack.add(-1);  
                    else if(nodeStack.pop() - 1 != 0) return false;  
                }  
  
                if(poll.right != null) {  
                    queue.add(poll.right);  
                    if(i <= size / 2) nodeStack.add(1);  
                    else if(nodeStack.pop() + 1 != 0) return false;  
                }  
  
            }  
  
            if(!valStack.isEmpty() || !nodeStack.isEmpty()) return false;  
  
        }  
        return true;  
    }  
}
```