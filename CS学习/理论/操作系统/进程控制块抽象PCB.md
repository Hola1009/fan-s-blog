计算机内部要管理任何现实事务, 都需要将其抽象成一组有关联的, 互为一体的数据, 在Java语言中, 我们可以通过类/对象来描述这一特征
```java
// 以下代码是 Java 代码的伪码形式，重在说明，无法直接运行
class PCB {
  // 进程的唯一标识 —— pid;
  // 进程关联的程序信息，例如哪个程序，加载到内存中的区域等
 // 分配给该资源使用的各个资源
  // 进度调度信息（留待下面讲解）
}
```
这样, 每一个PCB对象, 就代表着一个实实在在运行着的程序, 也就是进程
操作系统在通过这种数据结构, 例如线性表, 搜索树等讲PCB对象组织起来, 方便管理时进行增删改查的操作