[牛牛队列成环_牛客题霸_牛客网 (nowcoder.com)](https://www.nowcoder.com/practice/38467f349b3a4db595f58d43fe64fcc7?tpId=354&tqId=10590523&ru=/exam/oj&qru=/ta/interview-202-top/question-ranking&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3D%25E7%25AE%2597%25E6%25B3%2595%25E7%25AF%2587%26topicId%3D354)

```java
public class Solution {
    public boolean hasCycle (ListNode head) {
        // write code here
        HashSet<Integer> set = new HashSet<>();
        while(head != null) {
            if(!set.add(head.val)) return true;
            head = head.next;
        }
        return false;
    }
}
```