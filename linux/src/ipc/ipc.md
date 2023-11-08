# IPC
IPC工具分类：
1. 通信：关注进程间的数据交换
2. 同步：关注进程和线程操作间的同步
3. 信号

| 工 具 类 型 | 用于识别对象的名称 | 用于在程序中引用对象的句柄 |
| -------- | --- | ---- |
|  管道<br><br>FIFO   | 没有名称 <br><br> 路径名 | 文件描述符 <br><br>文件描述符 |
|UNIX domain socket<br><br>Internet domain socket|路径名<br><br>IP地址+端口号|文件描述符<br><br>文件描述符|
|System V 消息队列<br><br>System V 信号量<br><br>System V 共享内存|System V IPC键<br><br>System V IPC键<br><br>System V IPC 键|System V IPC 标识符<br><br>System V IPC 标识符<br><br>System V IPC 标识符|
|POSIX 消息队列<br><br>POSIX 命名信号量<br><br>POSIX 无名信号量<br><br>POSIX 共享内存|POSIX IPC 路径名<br><br>POSIX IPC 路径名<br><br>没有名称<br><br>POSIX IPC 路径名|`mqd_t` (消息队列描述符)<br><br>`sem_t *` (信号量指针)<br><br> `sem_t *` (信号量指针)<br><br>文件描述符|
|匿名映射内<br><br>存映射文件|没有名称<br><br>路径名|无<br><br>文件描述符|
|`flock()`文件锁<br><br>`fcntl()`文件锁|路径名<br><br>路径名|文件描述符<br><br>文件描述符|

## 总结
