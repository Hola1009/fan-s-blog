[删除链表峰值](https://www.nowcoder.com/share/jump/9977698741712405617967)

维护一个变量, 保存前驱节点, 每次遍历比较该节点的值与前驱与后继的值
符合条件执行操作
```java
public class Solution {
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     *
     * @param head ListNode类
     * @return ListNode类
     */
    public ListNode deleteNodes (ListNode head) {
        ListNode dummy = head;
        ListNode pre = null;
        while(head != null) {
            if(pre != null && head.next != null) {
                if(head.val > pre.val && head.val > head.next.val) {
                    pre.next = head.next;
                    head = head.next;
                }
            }
            pre = head;
            head = head.next;
        }
        return dummy;
    }
}
```
