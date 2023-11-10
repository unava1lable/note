## 进程的终止`_exit`和`exit`
通常，进程有两种终止方式。其一为异常（abnormal）终止，如由对一信号的接收而引发，该信号的默认动作为终止当前进程，可能产生核心转储（core dump）。此外，进程可使用_exit()系统调用正常（normally）终止。
```c
#include <unistd.h>

void _exit(int status);
```
`_exit()`的`status`参数定义了进程的终止状态（termination status），父进程可调用`wait()`以获取该状态。虽然将其定义为int类型，但仅有低8位可为父进程所用。按照惯例，终止状态为`0`表示进程“功成身退”，而非0值则表示进程因异常而退出。对非0返回值的解释则并无定例；不同的应用程序自成一派，并会在文档中加以描述。
调用`_exit()`的程序总会成功终止，即`_exit()`从不返回

程序一般不会直接调用`_exit()`，而是调用库函数`exit()`，它会在调用`_exit()`前执行各种动作。
```c
#include <stdlib.h>

void exit(int status);
```
`exit()`会执行的动作如下：
* 调用退出处理程序（通过`atexit()`和`on_exit()`注册的函数），其执行顺序与注册顺序相反。
* 刷新stdio流缓冲区
* 使用由`status`提供的值执行`_exit()`系统调用

程序的另一种终止方法是从`main()`函数中返回（return）。`return n == exit(n)`，因为调用`main()`的运行时函数会将`main()`的返回值作为`exit()`的参数。