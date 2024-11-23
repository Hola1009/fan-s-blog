这些笔记将临时记录我备赛 c++ 组所需要的知识, 所以并不体系

## 计算字符串长度的方法
```cpp
# include <iostream>
# include <cstring> // for strlen() function
int main() {
	using namespace std;
	char name[16] = "c++ Primer Plus";
	name[0] = 'C';
	cout << strlen(name) << endl; // ==> 15
}
```

## 输入

- 使用 `cin >> 变量名` 来进行输入, 并且只能读取一个单词()
```cpp
# include<iostream>

int main() {
	using namespace std;
	string name;
	cout << "Please enter your name : ";
	cin >> name;
	cout << endl << name << endl;
	return 0;
}
```

- 使用 `cin.getline(charArr, length)` 来读取一行数据, 
```cpp
# include<iostream>

int main() {
	using namespace std;
	char name[20];
	cout << "Please enter your name : ";
	cin.getline(name, 20);
	cout << endl << name << endl;
	return 0;
}
```

## string 
要使用 string 要导`# include <string>` 这个库, `using namespace std`

### string 类的一些操作
- strcpy(param1, param2) 第一个参数只能为字符串数组, 第二个参数只能为字符串数组或字符串字面量不能为字符串变量
- strcat() 合并操作, 将第二个参数的值追加到第一个参数的值, 同样的注意事项不能操作字符串变量

### stirng io
之前提到读取一行到字符数组中的操作是 `cin.getline`
而读取到字符串中的操作是 `getline(cin, str)`
```cpp
int main() {
	using namespace std;
	string str1;
	getline(cin, str1); // abc
	cout << str1; // ==> abc
	
	return 0;
}
```

## 指针
之前学到的知识我就不记录了
### new 关键字
用来分配内存的, 下面这段代码的语义是分配一片int大小的空间, 让指针pn指向这片空间, 且这个int 类型的数据 只能通过 \*pn 来访问呢
```cpp
int * pn = new int;
```

### delete 释放内存
用完的内存 可以使用 delete 来进行释放
```cpp
int * pn = new int;
delete pn
```
delete 只能用来释放 `new` 出来的内存块, 这意味着下面的操作是不被允许的
```cpp
int value = 1;
int * p = &value;
delelte p; // not allowed
```

### 使用 new 创建动态数组
很容易触类旁通的想到这样写, psome 指向的是数组的第一个元素, 访问其中的元素 `psome[n]` 这样操作 
```cpp
int * psome = new int[20];
```

删除的时候也需要这样操作
```cpp
delete [] psome;
```

> [!info] 对指针变量进行 + 1操作
> 增加的量等于其指向类型的字节数





