[1.数组分割 - 蓝桥云课 (lanqiao.cn)](https://www.lanqiao.cn/problems/3535/learning/?page=1&first_category_id=1&second_category_id=3&tags=2023&name=%E6%95%B0%E7%BB%84)

```java
import java.awt.Checkbox;
import java.util.Scanner;

public class Test {
	public static void main(String[] args) {
		int cnt = 0;
		for(int i = 1; cnt < 2023; i++) {
			if (check(i)) cnt++; 
			if (cnt == 2023) System.out.println(i);
		}
	}
	
	public static boolean check(int num) {
		
		int divide = 0, mod = 1;
		while(num / mod > 0) {
			divide += (num / mod) % 10;
			mod *= 10;
		}
		if (num % divide != 0) return false;
		//转成二进制判断
		divide = 0;
		mod = 1;
		while(num / mod > 0) {
			divide += (num / mod) % 2;
			mod *= 2;
		}
		if (num % divide != 0) return false;
		//转成八进制判断
		divide = 0;
		mod = 1;
		while(num / mod > 0) {
			divide += (num / mod) % 8;
			mod *= 8;
		}
		if (num % divide != 0) return false;
		//转成十六进制判断
		divide = 0;
		mod = 1;
		while(num / mod > 0) {
			divide += (num / mod) % 16;
			mod *= 16;
		}
		if (num % divide != 0) return false;
		return true;
	}
}
```