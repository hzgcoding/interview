# interview
# 网易互娱面试总结
## 现场二面技术总监面试
这个人看起来挺和善的，实际上还是有一套的，让你写代码，然后眼睛编译给你指出错误，然后修改，然后他就去玩手机了...
1. linux env multi-thread gdb debug 多线程调试 (回答对一半)
	首先介绍下基本命令
	- `info threads` 显示当前可以调试的所有线程，gdb会为每一个线程分配一个唯一ID，利用这个唯一Id可以切换到这个线程上下文环境中，并且前面有`*`标识的是当前调试的线程
		![](http://genge.cc/wp-content/uploads/2018/09/cfd3fadfb7b7fd387f68feb079e1a99a.png)
	- `thread ID` 切换调试的线程为指定ID的线程。这个Id是gdb为每个线程分配的，并不是操作系统分配的PID，可以通过`info threads`命令查看
	- `break xx.cpp:123 thread all` 在所有线程中相应的行上设置断点
	- `break apply ID1 ID2 cmd` 让一个或者多个线程执行gdb命令cmd
	- `break apply all cmd` 让所有被调试线程执行GDB命令command
	- `set print thread-events` **设置线程创建提醒**  当运行过程总产生新线程的时候会打印
	- `set scheduler-locking off|on|step` 这个是重点，经常被问到，在使用step或者continue命令调试当前被调试线程的时候，其他线程也是同时执行的，怎么只让被调试程序执行呢？通过这个命令就可以实现这个需求。
	- off 不锁定任何线程，也就是所有线程都执行，这是默认值。
	- on 只有当前被调试程序会执行。.
	- step 在单步的时候，除了next过一个函数的情况(熟悉情况的人可能知道，这其实是一个设置断点然后continue的行为)以外，只有当前线程会执行。该模式是对single-stepping模式的优化。此模式会阻止其他线程在当前线程单步调试时，抢占当前线。因此调试的焦点不会被以外的改变。其他线程不可能抢占当前的调试线程。其他线程只有下列情况下会重新获得运行的机会：当你‘next’一个函数调用的时候。当你使用诸如‘continue’、‘until‘、’finish‘命令的时候。其他线程遇到设置好的断点的时候。
其他GDB知识延伸
- 调试C++或者C的宏
	在编译程序的时候，加上`-ggdb3`参数，这样就可以调试宏
	- `info macro –` 你可以查看这个宏在哪些文件里被引用了，以及宏定义是什么样的。
	- `macro – `你可以查看宏展开的样子。
- 关联源文件
	- 如果在编译情况下加上`-g`参数，那么就可以包含debug信息，否者gdb找不到符号表
	- 可以使用`directory`命令来设置源文件的目录
		![](http://genge.cc/wp-content/uploads/2018/09/942cbce8346b6e1ef65a32ea8c1919a9.png)
- 条件断点
	基本语法` break  [where] if [condition]` 尤其是在一个循环或递归中，或是要监视某个变量。注意，这个设置是在GDB中的，只不过每经过那个断点时GDB会帮你检查一下条件是否满足。
- 添加参数
	1. gdb命令行的 –args 参数
	2. gdb环境中 set args命令。
- 设置变量
	1. 可以直接使用set命令 设置上下文环境变量值，可以模拟一些很难在测试中出现的情况，以防未来程序出错
	2. 声明变量，然后使用，语法为`$name = 1`
- X命令
	平时我们一般使用p命令打印参数值，但是这个命令必须指定变量名，不知道变量名的时候，我们可以使用X命令
	1. x/x 以十六进制输出
	2. x/d 以十进制输出
	3. x/c 以单字符输出
	4. x/i  反汇编 – 通常，我们会使用 x/10i $ip-20 来查看当前的汇编（$ip是指令寄存器）
	5. x/s 以字符串输出
- command命令  把一组命令录制下来打包成‘宏’
	![](http://genge.cc/wp-content/uploads/2018/09/1bc671b5484282a324ff2d3b68863cb7.png)

2. 循环队列判空 (OK)
	有三种方式处理这种问题
	1. 队列Queue结构中保存一个计数器count表示当前队列元素个数（最简单粗暴），但count等于队列cap的时候就队列满，count为0的时候队列空
	2. **少用一个元素空间**，约定以“队列头指针front在队尾指针rear的下一个位置上”作为队列“满”状态的标志。这种方法比较常用，但是面试官不让用.... front指向队首元素，rear指向队尾元素的下一个元素。即：
		- 队空时： front=rear
		- 队满时： (rear+1)%maxsize=front
	3. 还有一个比较取巧的办法，优化第一种方案：使用一个状态flag变量，初始值为0，但入队成功置flag = 1，当出队成功设置flag = 0。我们可以使用 `front == rear && flag` 表示队列满（在入队操作之后导致front=rear），可以使用`front == rear && ！flag`表示队列空（出队后导致f==r，显然是队列空）
	
3. 单向队列反转 (OK) 很简单
```cpp
struct Node
{
    int data;
    Node * next;
};

Node* reverse_list(Node* head)
{
    if(head == nullptr) return head;

    Node* node = nullptr;
    node = head;
    head = head->next;
    node = nullptr;

    while(head != nullptr)
    {
        Node* next = head->next;
        head->next = node;
        node = head;
        head = next;
    }

    return node;
}
```
## 现场面一面回忆总结
估计是小组长之类的面试官吧，去之前我还特意看下自己的衣装是否整洁，这个面试官感觉是从`工地`上回来的，衣服上很脏，典型程序员面孔，他问的问题算是比较全面
操作系统（linux）、数据库（mysql）、算法、数据结构、计算机网络基础、网络编程、语言基础（C++语言）、并发、以及具体业务设计，还有项目基本介绍，游戏服务器架构简单介绍
