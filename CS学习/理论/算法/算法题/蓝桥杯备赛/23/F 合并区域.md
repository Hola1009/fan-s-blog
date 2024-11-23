[1.合并区域 - 蓝桥云课 (lanqiao.cn)](https://www.lanqiao.cn/problems/3538/learning/?page=1&first_category_id=1&name=%E5%90%88%E5%B9%B6&tags=2023)

因为连接使原本不连接的区域连接起来了这种情况没有解决


贴个半成品先

```java
import java.util.ArrayDeque;  
import java.util.Queue;  
import java.util.Scanner;  
  
/**  
 * @author Fancier  
 * @version 1.0  
 * @description: TODO  
 * @date 2024/4/8 20:12  
 */  
public class Main {  
    static int[][] dir = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}, };  
    public static void main(String[] args) {  
        Scanner cin = new Scanner(System.in);  
        int n = cin.nextInt();  
        int[][] area_A = new int[n + 2][n + 2];  
        int[][] area_B = new int[n + 2][n + 2];  
        for (int i = 1; i <= n; i++) {  
            for (int j = 1; j <= n; j++) {  
                area_A[i][j] = cin.nextInt();  
            }  
        }  
  
        for (int i = 1; i <= n; i++) {  
            for (int j = 1; j <= n; j++) {  
                area_B[i][j] = cin.nextInt();  
            }  
        }  
        int merge = count(area_A, n) + count(area_B, n);  
        int max_A = maxArea(area_A, n);  
        int max_B = maxArea(area_B, n);  
  
        System.out.println(Math.max(merge, Math.max(max_A, max_B)));  
    }  
  
    /**  
     * 求单块土地里面的最大面积  
     * @param areaA  
     * @param n  
     * @return  
     */  
    private static int maxArea(int[][] areaA, int n) {  
        int res = 0;  
        for(int i = 2; i <= n - 1; i++) {  
            for (int j = 2; j <= n - 1; j++) {  
                if (areaA[i][j] == 1) {  
                   res = Math.max(res, bfs(areaA, new Point(i, j)));  
                }  
            }  
        }  
        return res;  
    }  
  
    /**  
     * 求边缘区域的最大面积  
     * @param area  
     * @param n  
     * @return  
     */  
  
    private static int count(int[][] area, int n) {  
        int res = 0;  
        for(int i = 1; i <= n; i++)  
            if (area[1][i] == 1)  
                res = Math.max(bfs(area, new Point(1, i)), res);  
        for(int i = 1; i <= n; i++)  
            if (area[i][1] == 1)  
                res = Math.max(bfs(area, new Point(i, 1)), res);  
        for(int i = 1; i <= n; i++)  
            if (area[n][i] == 1)  
                res = Math.max(bfs(area, new Point(n, i)), res);  
        for(int i = 1; i <= n; i++)  
            if (area[i][n] == 1)  
                res = Math.max(bfs(area, new Point(i, n)), res);  
        return res;  
    }  
  
    /**  
     * 广度优先求  
     * @param area  
     * @param point  
     * @return  
     */  
    private static int bfs(int[][] area, Point point) {  
        Queue<Point> queue = new ArrayDeque<>();  
        queue.add(point);  
        int res = 0;  
        while (!queue.isEmpty()) {  
            int size = queue.size();  
            for (int i = 0; i < size; i++) {  
                Point poll = queue.poll();  
                area[poll.x][poll.y] = 2;  
                res++;  
                for (int j = 0; j < 4; j++) {  
                    if (area[poll.x + dir[j][0]][poll.y + dir[j][1]] == 1)  
                        queue.add(new Point(poll.x + dir[j][0], poll.y + dir[j][1]));  
                }  
            }  
        }  
        return res;  
    }  
  
}  
class Point {  
    int x;  
    int y;  
  
    public Point(int x, int y) {  
        this.x = x;  
        this.y = y;  
    }  
}
```