[牛群分隔_牛客题霸_牛客网 (nowcoder.com)](https://www.nowcoder.com/practice/16d9dc3de2104fcaa52679ea796e638e?tpId=354&tqId=10591416&ru=/exam/oj&qru=/ta/interview-202-top/question-ranking&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3D%25E7%25AE%2597%25E6%25B3%2595%25E7%25AF%2587%26topicId%3D354)

分别用两个链表来保存 值小于 x  的 节点 和 值 大于等于 x 的结点
最后将两个链表合并即可

```java
public class Solution {
    public ListNode cow_partition (ListNode head, int x) {
        ListNode xNode = new ListNode(0);
        ListNode dummy = new ListNode(0);
        ListNode cur_1 = dummy;
        ListNode cur_2 = xNode;
        dummy.next = head;
        while(head != null) {
            ListNode temp = head;
            head = head.next;
            temp.next = null;
            if(temp.val < x) {
                cur_1.next = temp;
                cur_1 = temp;
            } else {
                cur_2.next = temp;
                cur_2 = temp;
            }
        }
        cur_1.next = xNode.next;
        return dummy.next;
    }
}
```