CPU中有些寄存器, 属于没有特定含义, 就只是用来保存运算的中间结果的, 还有些寄存器, 是由特定含义, 特定作用的
1. 保存当前执行到哪个指令(程序计数器)
	是一个2字节/4字节/8字节 整数
	这个整数存的是一个内存地址
2. 维护栈相关的寄存器
	 通过这一组(一组是两个)维护当前程序的"调用栈"
	 栈, 也是一块内存, 这个内存里就保存当前这个程序方法调用过程中的一系列关系(包含局部变量和方法参数)
	 edp始终指向栈底
	 esp始终指向栈顶, 修改esp的值就可以实现"入栈"/"出栈"
		 (push指令完成上述操作)
	 有了这个 才知道一个方法执行完毕后, 回到哪里执行
3. 其他的通用寄存器, 往往是用来保存计算的中间结果的