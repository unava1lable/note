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

// 删除一个消息队列名，并当所有进程都关闭该队列时标记它一遍删除
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
## 总结
