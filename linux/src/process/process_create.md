## 创建新进程`fork()`
系统调用`fork()`创建一新进程（child），几近于对调用进程（parent）的翻版
```c
#include <unistd.h>


// 在父进程中: 返回子进程ID
// 在子进程中: 返回0
// 出错时返回-1
pid_t fork(void);
```
执行`fork()`时，子进程会获得父进程所有文件描述符的副本。这些副本的创建方式类似于`dup()`，这也意味着父、子进程中对应的描述符均指向相同的打开文件句柄。

## `fork()`的内存语义
从概念上说来，可以将`fork()`认作对父进程程序段、数据段、堆段以及栈段创建拷贝。在一些早期的 UNIX 实现中，此类复制确实是原汁原味：将父进程内存拷贝至交换空间，以此创建新进程映像（image），而在父进程保持自身内存的同时，将换出映像置为子进程。不过，真要是简单地将父进程虚拟内存页拷贝到新的子进程，那就太浪费了。原因有很多，其中之一是：`fork()`之后常常伴随着`exec()`, 这会用新程序替换进程的代码段，并重新初始化其数据段、堆段和栈段。大部分现代 UNIX 实现（包括 Linux）采用两种技术来避免这种浪费。
1. 内核将每一进程的代码段标记为只读，从而使进程无法修改自身代码。这样，父、子进程可共享同一代码段。系统调用 `fork()`在为子进程创建代码段时，其所构建的一系列进程级页表项均指向与父进程相同的物理内存页帧。
2. 对于父进程数据段、堆段和栈段中的各页，内核采用写时复制（copy-on-write）技术来处理。最初，内核做了一些设置，令这些段的页表项指向与父进程相同的物理内存页，并将这些页面自身标记为只读。调用`fork()`之后，内核会捕获所有父进程或子进程针对这些页面的修改企图，并为将要修改的页面创建拷贝。系统将新的页面拷贝分配给遭内核捕获的进程，还会对子进程的相应页表项做适当调整。从这一刻起，父、子进程可以分别修改各自的页拷贝，不再相互影响。

## `vfork()`
类似于`fork()`，`vfork()`可以为调用进程创建一个新的子进程。然而，`vfork()`是为子进程立即执行`exec()`的程序而专门设计的。
```c
#include <unistd.h>

pid_t vfork(void);
```
因为：
1. 无需为子进程复制虚拟内存页或页表。相反，子进程共享父进程的内存，直至其成功执行了`exec()`或是调用`_exit()`退出。
2. 在子进程调用`exec()`或`_exit()`之前，将暂停执行父进程。

`vfork`比`fork`更有效率。由于子进程使用父进程的内存，因此子进程对数据段、堆或栈的任何改变将在父进程恢复执行时为其所见。此外，如果子进程在`vfork()`与后续的`exec()`或`_exit()`之间执行了函数返回，这同样会影响到父进程。

> SUSv3 指出，在如下情况下程序行为未定义：a）修改了除用于存储`vfork()`返回值的`pid_t`型变量之外的任何数据；b）从调用`vfork()`的函数中返回；c）在成功地调用`_exit()`或执行`exec()`之前，调用了任何其他函数。