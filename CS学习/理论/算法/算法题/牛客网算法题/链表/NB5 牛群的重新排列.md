[牛群的重新排列](https://www.nowcoder.com/share/jump/9977698741712460106022)

```java
public class Solution {
    public ListNode reverseBetween (ListNode head, int left, int right) {
        // write code here
        if(left == right) return head;
        ListNode cur = new ListNode(501);
        cur.next = head;
        ListNode start = cur;
        ListNode end = null;
        head = cur;
        int index = 1;
        while(cur != null) {
            if(index == left)
                start = cur;
            if(index == right + 1) {
                end = cur.next;
                break;
            }
            index++;
            cur =  cur.next;
        }
        reverse(start.next, right - left, start).next = end;
        return head.next;
    }
    public ListNode reverse(ListNode node, int end, ListNode start) {
        if(end == 0) {
            start.next = node;
            return node;
        }
        reverse(node.next, end - 1, start).next = node;
        return node;
    }
}
```