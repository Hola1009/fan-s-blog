[1.蜗牛 - 蓝桥云课 (lanqiao.cn)](https://www.lanqiao.cn/problems/4985/learning/?page=1&first_category_id=1&name=%E8%9C%97%E7%89%9B)

用dp来做
每个柱子两种状态, 一种是从最后在传送点入口上, 一种是在底下

传送点底下 是由 上一个柱子底下或者传送点入口转过来的

传送点入口 是由传送点出口转过来的

dp\[i]\[0] = min(dp\[i - 1]\[0] + x\[i] - x\[i - 1],  dp\[i - 1]\[1] + 1.3 \* b\[i]);
dp\[i]\[1] = min(dp\[i - 1]\[1] + xxx, dp\[i - 1]\[0] + x\[i] - x\[i - 1] + 0.7 \* b\[i])

```java
import java.util.Scanner;  
  
/**  
 * @author Fancier  
 * @version 1.0  
 * @description: TODO  
 * @date 2024/4/8 19:20  
 */  
public class Main {  
    public static void main(String[] args) {  
        Scanner cin = new Scanner(System.in);  
        int n = cin.nextInt();  
        int[] x = new int[n];  
        int[] a = new int[n];  
        int[] b = new int[n];  
        for (int i = 0; i < n; i++) x[i] = cin.nextInt();  
  
        for (int i = 0; i < n - 1; i++) {  
            a[i] = cin.nextInt();  
            b[i + 1] = cin.nextInt();  
        }  
  
        double[][] dp = new double[n][2];  
          
        dp[0][0] = x[0];  
        dp[0][1] = x[0] + a[0] / 0.7;  
  
        for (int i = 1; i < n - 1; i++) {  
            dp[i][0] = Math.min(dp[i - 1][0] + x[i] - x[i - 1], dp[i - 1][1] + b[i] / 1.3);  
            double t = (a[i] - b[i]) / 0.7;  
            if (a[i] < b[i]) t = (b[i] - a[i]) / 1.3;  
            dp[i][1] = Math.min(dp[i - 1][1] + t, dp[i][0] + a[i] / 0.7);  
        }  
  
        dp[n - 1][0] = Math.min(dp[n - 2][0] + x[n - 1] - x[n - 2], dp[n - 2][1] + b[n - 1] / 1.3);  
  
        System.out.printf("%.2f", dp[n - 1][0]);  
    }  
}
```
