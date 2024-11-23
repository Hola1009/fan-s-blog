## 原题链接
[牛的品种排序IV_牛客题霸_牛客网 (nowcoder.com)](https://www.nowcoder.com/practice/bd828af269cd493c86cc915389b02b9f?tpId=354&tqId=10595837&ru=/exam/company&qru=/ta/interview-202-top/question-ranking&sourceUrl=%2Fexam%2Fcompany)

## 思路

弄两个新的链表sHead 和 bHead, 遍历传入的链表, 将值为0的节点链接到sHead尾部, 将值为1的节点链接到bHead尾部; 遍历完后, 再把两个链表拼接起来就成
## 贴个代码
```java
public class Solution {
    public ListNode sortCowsIV (ListNode head) {
        ListNode sHead = new ListNode(0);
        ListNode bHead = new ListNode(0);
        ListNode cur_1 = sHead;
        ListNode cur_2 = bHead;
        while(head != null) {
            ListNode temp = head.next;
            head.next = null;
            if(head.val == 1) {
                cur_2.next = head;
                cur_2  = cur_2.next;
            } else {
                cur_1.next = head;
                cur_1 = cur_1.next;
            }
            head = temp;
        }
        cur_1.next = bHead.next;
        return sHead.next;
    }
}
```
---


具体代码参上

好的!本次分享到这就结束了
如果对铁汁你有帮助的话，记得点赞👍+收藏⭐️+关注➕
我在这先行拜谢了：）
