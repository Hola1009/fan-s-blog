[牛群排列去重](https://www.nowcoder.com/share/jump/9977698741712406711915)
```java
public class Solution {
    public ListNode deleteDuplicates (ListNode head) {
        boolean[] map = new boolean[201];
        ListNode cur = head;
        ListNode pre = null;
        while(cur != null) {
            if(map[cur.val]) {
                pre.next = cur.next;
            } else {
                pre = cur;
            }
            map[cur.val] = true;
            cur = cur.next;
        }
        return head;
    }
}
```
