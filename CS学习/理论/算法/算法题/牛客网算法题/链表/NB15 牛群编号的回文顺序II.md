# 原题链接
[牛群编号的回文顺序II_牛客题霸_牛客网 (nowcoder.com)](https://www.nowcoder.com/practice/e887280579bb4d91934378ea359f632e?tpId=354&tqId=10595827&ru=/exam/oj&qru=/ta/interview-202-top/question-ranking&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3D%25E7%25AE%2597%25E6%25B3%2595%25E7%25AF%2587%26topicId%3D354)
# 一种可行的思路
这道题是 NB14 的升级, 大家可以看看我关于 NB 14 的题解[NB14 牛群编号的回文顺序](http://t.csdnimg.cn/IzQYJ)

先遍历链表, 将节点的值(1-9)用 StringBuffer 给存起来, 再用一个list来存每个节点
## 用动态规划来解题
然后再用 dp 来解题
填表的时候 更新最长回文子串的起始下标和结束下标

填完表后, 看看这个最长字串的长度是否和原来的链表一样长, 是就返回空

否则 把ist结束下标上的节点指向空

再返回起始下标上的节点
### 状态转移方程为: 
dp\[i]\[j] = dp\[i + 1]\[j - 1] && strB\[i] == strB\[j] (i > j + 1)
dp\[i]\[j] = true; (i = j)
dp\[i]\[j] = strB\[i] == strB\[j] (i + 1 = j)

### 填表顺序
因为 (i + 1, j - 1) 在 (i, j) 的左下角, 而且 i 必然不大于 j 所以我们 从左上到右下 斜着填表
```
\>   \>
 \>   \>
  \>   \>
```
# 贴个代码

```java
public class Solution {
    public ListNode maxPalindrome (ListNode head) {
        List<ListNode> list = new ArrayList<>();
        StringBuffer strB = new StringBuffer();
        int start = -1, end = -1;
        while (head != null) {
            strB.append(head.val);
            list.add(head);
            head = head.next;
        }
        int len = strB.length();
        int maxLen = 0;
        boolean[][] dp = new boolean[len][len];
        dp[0][0] = true;
        for (int k = 0; k < len; k++) {
            for (int i = 0; i + k < len; i++) {
                int j = i + k;
                if (i == j) dp[i][j] = true;
                else if (i + 1 == j) dp[i][j] = strB.charAt(i) == strB.charAt(j);
                else dp[i][j] = strB.charAt(i) == strB.charAt(j) && dp[i + 1][j - 1];
                if(dp[i][j] && j - i + 1 > maxLen) {
                    start = i;
                    end = j;    
                }
            }
        }
        if(end - start + 1 == list.size()) return null;
        list.get(end).next = null;
        return list.get(start);
    }
}
```