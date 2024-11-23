[牛群的重新分组](https://www.nowcoder.com/share/jump/9977698741712457442348)

```java
public class Solution {
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     *
     * @param head ListNode类
     * @param k int整型
     * @return ListNode类
     */
    public ListNode reverseKGroup (ListNode head, int k) {
        if(k == 1) return head;
        // write code here
        List<ListNode> list = new ArrayList<>();
        int cnt = 0;
        ListNode pre = new ListNode(0);
        ListNode cur = head;
        pre.next = head;
        while(cur != null) {
            cnt++;
            list.add(cur);
            cur = cur.next;
            if(cnt % k == 0) {
                int i;
                for(i = list.size() - 1; i > list.size() - k; i--) {
                    list.get(i).next = list.get(i - 1);
                }
                list.get(i).next = cur;
                pre.next = list.get(list.size() - 1);
                pre = list.get(i);
                cnt = 0;
            }
        }
        return list.get(k - 1);
    }
}
```