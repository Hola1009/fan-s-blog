[牛群的能量值_牛客题霸_牛客网 (nowcoder.com)](https://www.nowcoder.com/practice/fc49a20f47ac431981ef17aee6bd7d15?tpId=354&tqId=10590540&ru=/exam/oj&qru=/ta/interview-202-top/question-ranking&sourceUrl=%2Fexam%2Foj)

模拟加法运算就可以解决这道题了, 题目还很贴心的告诉说链表是逆序的, 这就极大的方便了我们操作了, 我们常加法就是从最后一位开始算的不是吗
先 个位相加, 再进位,
十位相加 + 进的哪一位 再进位
百位相加 + 进的哪一位 再进位
...
如此循环便可得到最终答案
```java
import java.util.*;

public class Solution {
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     *
     * @param l1 ListNode类
     * @param l2 ListNode类
     * @return ListNode类
     */
    public ListNode addEnergyValues (ListNode l1, ListNode l2) {
        // write code here
        ListNode head = new ListNode(0);
        ListNode pre = head;
        int rem = 0;
        ListNode cur_1 = l1;
        ListNode cur_2 = l2;
        while(cur_1 != null || cur_2 != null || rem != 0) {
            int a = cur_1 == null ? 0 : cur_1.val;
            int b = cur_2 == null ? 0 : cur_2.val;
            pre.next = new ListNode((a + b + rem) % 10);
            rem = (a + b + rem) / 10;
            pre = pre.next;
            cur_1 = cur_1 == null ? null : cur_1.next;
            cur_2 = cur_2 == null ? null : cur_2.next;
        }
        return head.next;
    }
}
```

---


具体代码参上

好的!本次分享到这就结束了
如果对铁汁你有帮助的话，记得点赞👍+收藏⭐️+关注➕
我在这先行拜谢了：）
