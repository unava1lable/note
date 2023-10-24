# IPC

## System V IPC
### System V IPC简介

## POSIX IPC
### POSIX IPC简介

### 消息队列
#### API介绍
```c
typedef int mqd_t;

struct mq_attr {
  long mq_flags;	/* Message queue flags.  */
  long mq_maxmsg;	/* 消息队列中消息的最大数量，必须大于0，无法修改  */
  long mq_msgsize;	/* 每条消息的最大大小，必须大于0，无法修改  */
  long mq_curmsgs;	/* Number of messages currently queued.  */
  long __pad[4];
};

// 创建一个新的或打开一个现有的消息队列，返回一个消息队列描述符
// 出错时返回-1
mqd_t mq_open(const char *name, int flag, ... /* mode_t mode, struct mq_attr *attr */);


// 向队列（mqdes）写入一条消息
// 出错时返回-1
int mq_send(mqd_t mqdes, const char *msg_ptr, size_t msg_len, unsigned int msg_prio);


// 从队列（mqdes）读取一条消息
// 出错时返回-1
ssize_t mq_receive(mqd_t mqdes, char *msg_ptr, size_t msg_len, unsigned int *msg_prio);


// 关闭一个消息队列
// 出错时返回-1
int mq_close(mqd_t mqdes);

// 删除一个消息队列名，并当所有进程都关闭该队列时删除
// 出错时返回-1
int mq_unlink(const char *name);


// 获取消息队列特性
// 出错时返回-1
int mq_getattr(mqd_t mqdes, struct mq_attr *mqstat);


// 修改消息队列特性
// 出错时返回-1
int mq_setattr(mqd_t mqdes, const struct mq_attr * mqstat, struct mq_attr * oldmqstat);


// 注册调用进程在一条消息进入空的消息队列（mqdes）时接收通知
// 出错时返回-1
int mq_notify(mqd_t mqdes, const struct sigevent *notification);
// 如果一个消息队列上已经存在注册进程了，则调用会失败且返回EBUSY
// 只有当一条新消息进入之前为空的队列时注册进程才会收到通知
// 当向注册进程发送了一个通知之后就会删除注册信息
// 一个进程可以通过在调用mq_notify(mqdes, NULL)来撤销自己在消息通知上的注册信息
```

#### 示例
1. 基本使用
```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <assert.h>
#include <mqueue.h>

int main(void) {
    mqd_t d = mq_open("/myq", O_CREAT | O_RDWR, S_IRUSR | S_IWUSR);
    char buffer[] = "hello\n";
    char receive[128];

    if (d < 0) {
        printf("open error\n");
        exit(1);
    }
    int pid = fork();

    assert(pid >= 0);

    if (pid == 0) {
        if (mq_send(d, buffer, sizeof(buffer), 0) < 0) {
            printf("send error\n");
            mq_unlink("/myq");
            exit(1);
        }
        exit(0);
    } else {
        if (mq_receive(d, receive, 128, NULL) < 0) {
            printf("receive error\n");
            mq_unlink("/myq");
            exit(1);
        }
    }
    printf("received: %s", receive);

    assert(mq_unlink("/myq") == 0);
    return 0;
}
```
### 信号量
POSIX信号量是一个不小于0的整数，如果一个进程试图将一个信号量的值减小到小于0，那么取决于所使用的函数，调用会阻塞或返回一个表明当前无法执行相应操作的错误。
#### API介绍
```c
#include <fcntl.h>
#include <sys/stat.h>
#include <semaphore.h>

#define SEM_FAILED      ((sem_t *) 0)

typedef union
{
  char __size[__SIZEOF_SEM_T];
  long int __align;
} sem_t;

// 原子地打开一个命名信号量
// 出错返回SEM_FAILED
sem_t *sem_open (const char *__name, int __oflag, ... /* mode_t mode, unsigned int value */);


// 递减（减少1）sem引用的信号量的值
// 出错时返回-1
int sem_wait (sem_t *sem);
//如果信号量的当前值大于0，立即返回。如果信号量的当前值等于0，则阻塞直到信号量的值大于0为止


// sem_wait的非阻塞版本，如果递减操作无法立即被执行，那么失败并返回EAGAIN
// 出错时返回-1
int sem_trywait (sem_t *sem);


// 递增（增加1）sem引用的信号量的值
// 出错时返回-1
int sem_post (sem_t *sem);


// 获取信号量当前的值
// 出错时返回-1
int sem_getvalue (sem_t *sem, int *sval);


// 关闭信号量，即释放系统为该进程关联到该信号量之上的所有资源，并递减引用该信号量的进程数
// 出错时返回-1
int sem_close (sem_t *sem);


// 删除一个命名信号量，并当所有进程都关闭它时删除
// 出错时返回-1
int sem_unlink (const char *name);


// 对未命名信号量进行初始化
// 出错时返回-1
int sem_init (sem_t *sem, int pshared, unsigned int value);
// pshared == 0：在线程中共享
// pshared != 0：在进程中共享


// 销毁一个未命名信号量
// 出错时返回-1
int sem_destroy (sem_t *sem);
```
#### 示例
1. 基本使用
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <wait.h>
#include <sys/stat.h>
#include <semaphore.h>

int main(void) {
    sem_t *sema;

    sema = sem_open("/mysem", O_CREAT | O_RDWR, S_IRUSR | S_IWUSR, 0);
    if (sema == SEM_FAILED) {
        printf("sem open failed\n");
        exit(1);
    }

    int pid = fork();
    if (pid == 0) {
        sem_wait(sema);
        printf("Always Last\n");
        sem_post(sema);
        exit(0);
    } else {
        printf("Always First\n");
        sem_post(sema);
        wait(NULL);
    }
    sem_close(sema);
    sem_unlink("/mysem");
    return 0;
}
```

### 共享内存
POSIX共享内存能够让无关进程共享一个映射区域而无需创建一个相应的映射文件。

要使用共享内存需要：
1. 使用`shm_open()`打开一个共享内存对象。
2. 将上一步获取的文件描述符传入mmap()，并在其`flags`参数中指定`MAP_SHARED`。
#### API介绍
```c
// 打开一个共享内存对象
// 出错时返回-1
int shm_open (const char *name, int oflag, mode_t mode);


// 删除一个共享内存对象
// 出错时返回-1
int shm_unlink (const char *name);
```
#### 示例

## 总结
