## STL
### STL 的诞生
- 长久以来, 软件界一直希望建立一种可重入利用的东西
- c++的面向对象和泛型编程思想, 目的就是复用性的提升
- 大多情况下, 数据结构和算法都未能有一套标准, 导致被迫从事大量重复工作
- 为了建立数据结构和算法的一套标准, 诞生了stl

### STL 基本概念
- STL 标准模板库
- STL 从广义上分为 `容器`, `算法`, `迭代器`
- STL 几乎所有的代码都采用了模板类或者模板函数

### STL 六大组件
大体分为: `容器`, `算法`, `迭代器`, `仿函数`, `适配器`, `空间配置器`

### string 
#### 构造函数
- string() 创建空字符串
- string(char* str) 使用字符串数组进行初始化
- string(const string& str) 使用 string 对象初始化另应给string对象
- string(int n, char c) 使用 n 个字符进行初始化

#### string 的赋值操作
- 直接用字符换数组赋值
- 直接用字符串对象赋值
- 直接用单个字符赋值
- `string& assigh(const char * s, int n)` 把字符数组的前n个字符赋给当前的字符串
- `string& assigh(const string & s, int n)` 把字符数串的前n个字符赋给当前的字符串
- `string& assigh(int n, char c)` 把n个字符赋给当前的字符串

#### 字符串拼接
- 直接拼接字符数组或字符串字面量
- 直接拼接字符
- 直接拼接字符串对象
- 使用 `string& append(const char * s)`
- 使用 `string& append(const char * s, int n)` 拼接字符串的前n个字符
- 使用 `string& append(const string &s)`
- 使用 `string& append(const string &s, int pos, int n)` 从指定位置开始拼接n个字符

#### string 替换和查找
##### 函数原型
**查找**
- `int find(const string& str, int pos = 0) const` 查找str第一次出现的位置, 从 pos 开始找
- `int find(const chat * str, int post` 查找str第一次出现的位置, 从 pos 开始找
- `int find(const char * s, int pos, int n) const` 从 pos 位置查找前n个字符第一次出现的位置
- `int find(const char c, int pos = 0) const` 查找字符c第一次出现的位置
- `int rfind(const string & str, int pos = npos) const` 查找str最后一次出现的位置, 从pos开始查找
- `int rfind(const char * str, int pos = npos)` 查找str最后一次出现的位置, 从pos开始查找(从右往左查)
- `int rfind(const char * str, int pos, int n) const` 从pos查找str的前n个字符最后出现的位置
- `int rfind(const char c, int pos = 0) const` 查找字符 c 最后一次出现的位置

**替换**
- `string & replace(int pos, int n, const string& str)` 替换 pos 开始, n 个字符为 str
- `string& replace(int pos, int n, const char * str)` 替换 pos 开始, n 个字符为 str

#### 字符串比较
比较方式是按照 ascII 码值进行比较

##### 函数原型
- `int compare(const string &s) const;` 与 s 字符串进行比较
- `int compare(const char * s) const;` 与 s 字符串进行比较

#### 字符串存取
string 中单个字符串存取方式有两种, 因为拿到的值是引用类型, 所以可以直接修改
- `char& operator[] (int n)` 通过 \[\] 方式去字符
- `char& at(int n)` 通过at方法获取字符

```cpp
int main() {
	string str = "asdfghjkl";
	cout << str[0] << endl; // ==> a
	str[0] = '1';
	cout << str << endl; // ==> 1sdfghjkl
}
```

#### 字符串插入和删除
对string字符串进行插入和删除操作

##### 函数原型
- `string& insert(int pos, const char * s);` 从指定位置开始插入字符串
- `string& insert(int pos, const string & s);`  从指定位置开始插入字符串
- `string& insert(int pos, int n, char c);` 在指定位置插入 n 个字符
- `string& erase(int pos, int n = npos)` 删除 pos 开始的 n 个字符

#### 子串获取

##### 函数原型
- `string substr(int post = 0, int n = npos)` 返回由pos开始的n个字符组成的字符串

### vector
vector 和 普通数组的区别, vector 可以动态扩展

##### 动态拓展
是将引用指向更大的内存空间(实现拷贝后)

#### 构造函数
- `vector<T> v;`
- `vector(v.begin(), v.end());` 把一个左闭右开的区间拷贝给本身
- `vector(n, elem);` 构造函数将 n 个 elem 拷贝给本身
- `vector(const vector & vec);` 拷贝构造

#### vector 赋值操作
- `vector& operator=(const vector &vec);` 重载等号操作符
- `assign(beg, end)` // 将 左闭右开 区间的值赋值给本身
- `assign(n, elem)` 将 n 个 elem 赋值给本身

#### vector 容量和大小
对 vector 容器的容量和大小操作

##### 函数原型
- `empty();` 判断容器是否为空
- `capacity();` 容器的容量
- `size()` 返回容器中元素的个数
- `resize(int len)` 重新指定容器长度
- `resize(int len, elem)` 重新指定容器长度, 容器边长, 用 elem 来填充新位置

#### vector插入和删除
对容器进行插入和删除

##### 函数原型
- `push_back(ele)` 尾插
- `pop_back();` 删除最后的元素
- `insert(const_iterator pos, ele);` 迭代器指向位置 pos 插入元素 ele
- `insert(const_iterator pos, int count, ele);` 在迭代器指向位置插入 count 个 ele
- `erase(const_iterator pos)` 删除迭代器指向位置的元素
- `erase(const_iterator start, const_iterator end)`  删除指定区间的元素(左闭右开)
- `clear();`

#### vector 数据存取

##### 函数原型
- `at(int idx)` 返回索引锁一象数据
- `operator[]` 返回索引idx所值的数据
- `front()` 第一个
- `back()` 最后一个

#### vector 互换容器

##### 函数原型
- swap(vec) 将 vec 与本身的元素互换

巧用 swap 收缩内存
```cpp
vector<int>(v).swap(v)
```
分析:
```cpp
vector<int>(v) // 拷贝v来创建一个匿名的容器对象
vector<int>(v).swap(v) // 用匿名容器与v进行交换
```

#### vector 预留空间
减少vector在动态拓展容量时的拓展次数

##### 函数原型
- `reserve(int len);` 容器预留len个元素长度, 预留位置不初始化, 元素不可访问
注意: 不要错误拼写成了 reverse

### deque 容器
描述: 双端队列, 可以对头段进行插入和删除操作

#### deque 和 vector 区别
- vector 对于头部的插入效率太低, 数据越大. 效率越低
- deque 相对而言, 对头部的插入删除速度比vector快
- vector访问元素时的速度会比 deque 快, 这和两者的内部实现有关

deque 内部原理:
的却内部有个中控器, 维护每段缓冲区中的内容, 缓冲区中存放真实数据, 中控器维护的时每个缓冲区的地址, 使得deque看起来像一片连续的内存空间, deque 也是支持随机访问的
![[Pasted image 20241021094626.png]]


#### 构造函数
- `deque<T> deqT;` 默认构造形式
- `deque<beg, end>;` 构造函数将左闭右开的区间拷贝给本身
- `deque(n, elem);` 将 n 个 elem 拷贝给本身
- `deque(const deque & deq)` 拷贝构造

#### 赋值操作
- `vector& operator=(const deque &deq);` 重载等号操作符
- `assign(beg, end)` // 将 左闭右开 区间的值赋值给本身
- `assign(n, elem)` 将 n 个 elem 赋值给本身

#### deque 大小操作

- `deque.empty();` 
- `deque.size()`
- `deque.resize(num)`
- `deque.resize(num, elem)`

#### deque 插入和删除
两端插入操作
- `push_back(elem);` 尾插
- `push_front(elem);` 头插
- `pop_back()`;
- `pop_front();`

指定位置操作
- `insert(pos, elem)`
- `insert(pos, n, elem)
- `insert(pos, beg, end)` 在pos位置插入一个区间的位置
- `clear();`
- `erase(beg, end);`
- `erase(pos);`

#### deque 数据存取
- `at(int idx);` 返回引用
- `operator[];` 返回引用
- front();
- back();

#### 排序
- `srot(iterator beg, iterator end);`

### stack 容器
栈容器, 不多介绍了

#### stack 构造方法
- `stack<T> stk`
- `stack(const stack & stk)` 拷贝构造

#### 赋值操作
`stack & operator = (const stack & stk)`

#### 数据存取
- `push(elem);` 
- `pop();`
- `top();`

#### 大小操作
- `empty()`
- `size()`

### queue 容器
队列容器不多介绍了
#### queue 构造方法
- `qeueu<T> que`
- `queue(const queue & que)` 拷贝构造

#### 赋值操作
`queue & operator = (const queue & que)`

#### 数据存取
- `push(elem);` 
- `pop();`
- `top();`
- `back();`

#### 大小操作
- `empty()`
- `size()`

### list 容器
链表容器, 不多介绍了

#### 构造函数
- `list<T> list;`
- `list(beg, end);`
- `list(n, elem);`
- `list(const list &lst);`

#### list 赋值和交换操作
- `assign(beg, end);`
- `assign(n, elem);`
- `list& operator=(const list & list);`
- `swap(lst);`

#### 大小操作
- `size();`
- `empty()`
- `resize(num)`
- `resize(num, elem)`

#### 插入和删除
- `push_back(elem);`
- `pop_back();`
- `push_front(elem);`
- `insert(pos, elem);`
- `insert(pos, n, elem);`
- `insert(pos, beg, end);`
- `clear();
- `erase(beg, end);`
- `erase(pos);`
- `remove(elem);`

#### 数据存取
- `front();`
- `back()`

> [!warning] 
> 迭代器支持 `++` `--` 这种操作, 但是不支持 `+ 1` 这种随机访问的方式


#### 翻转和排序
- `reverse()`
- `sort()`
不支持随机访问的容器不支持标准的 `sort()` ,容器内部会提供排序, sort可以接收自定义的比较方法

### set / multiset 容器
在插入的时候就会进行排序

set 和 multiset 的区别, set 不允许存放重复元素, multiset 允许

#### set删除
`erase(elem)` 可以直接通过传值的方式进行删除

#### set 查找和统计
- `find(key)` 查找key是否存在, 返回迭代器, 没找到会返回set.end()
- `count(key)` 统计key的元素个数

