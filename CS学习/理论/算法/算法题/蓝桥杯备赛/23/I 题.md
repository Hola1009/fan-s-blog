
这是一个背包问题
例子分析
```d
4 2
-4 10
-2 7
```
公式 $Hi(X) = max (Ki × X^2 + Bi × X,   0)$

 尽可能使 Ki × X + Bi 大 ?

第一个
1  2  3  
6  4  0

第二个
1   2   3
5   6   3

所以 破题点在求极值点?
$aX^2+bX+c$

$$
顶点公式: \frac{b}{-2a}
$$



第一个的 顶点为 10 / 8
向下取整为 1, 向上取整为2

6 1 性价比为 6
6 2性价比为 3

优先选性价比高的

6 + 6 

如果项目没选完 剩下的人不足以达到顶点咋整, 

-1x^2 + 5x

      人数   最大开支   
-5 7    1        2   
-2 8    2        8  
-3 7    1         4   
-1 5    2         6   


