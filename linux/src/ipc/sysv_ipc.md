## System V IPC简介

## 消息队列
### API介绍
```c
#include <sys/types.h>
#include <sys/msg.h>

struct msqid_ds {
    struct ipc_perm msg_perm;	        /* ownership and permission */
    time_t msg_stime;		            /* time of last msgsnd command */
    time_t msg_rtime;		            /* time of last msgrcv command */
    time_t msg_ctime;		            /* time of last change */
    unsigned long __msg_cbytes;         /* current number of bytes on queue */
    msgqnum_t msg_qnum;		            /* number of messages currently on queue */
    msglen_t msg_qbytes;		        /* max number of bytes allowed on queue */
    pid_t msg_lspid;		            /* pid of last msgsnd() */
    pid_t msg_lrpid;		            /* pid of last msgrcv() */
    unsigned long __glibc_reserved4;
    unsigned long __glibc_reserved5;
};


// 创建一个新的消息队列或获取一个既有队列的标识符
// key: IPC_PRIVATE或ftok()返回的一个键
// msgflg: 权限，IPC_CREAT和IPC_EXCL
// 出错时返回-1
int msgget (key_t key, int msgflg);


// 向消息队列写入一条消息
// msgp: struct {
//     long mtype;      消息类型，大于0
//     char mtext[];    消息体
// }
// msgsz: mtext包含的字节数
// msgflg: 目前只有IPC_NOWAIT
// 出错时返回-1
int msgsnd (int msqid, const void *msgp, size_t msgsz, int msgflg);

// 从消息队列中读取并删除一条消息，并将内润复制到msgp指向的缓冲区中
// msgtyp: == 0，删除并返回第一条消息
//          > 0，删除并返回第一条msgtyp == mtype的消息
//          < 0，将等待消息当成优先队列来处理。mtype最小并且 <= |msgtyp|的第一条消息会被删除并返回
// 出错时返回-1
ssize_t msgrcv (int msqid, void *msgp, size_t msgsz, long int msgtyp, int msgflg);

// 执行控制操作
// cmd: IPC_RMID，删除消息队列及msqid_ds。队列中所有消息都会丢失，所有被阻塞的读者和写者进程会立即醒来，忽略buf参数
//      IPC_STAT，将与这个消息队列的msqid_ds数据结构的副本放到buf指向的缓冲区中
//      IPC_SET，使用buf指向的缓冲区提供的值更新与这个消息队列关联的msqid_ds数据结构中被选中的字段
// 出错时返回-1
int msgctl (int msqid, int cmd, struct msqid_ds *buf);
```

### 示例
1. 基本使用
```c
#include <sys/types.h>
#include <sys/msg.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct msg {
    long mtype;
    char mtext[16];
};

int main(void) {
    struct msg m, q;
    m.mtype = 1;
    memcpy(m.mtext, "hello\n", 7);

    int f = msgget(IPC_PRIVATE, S_IRUSR | S_IWUSR | IPC_CREAT);
    if (f < 0) {
        printf("get error\n");
        exit(1);
    }

    msgsnd(f, &m, 7, IPC_NOWAIT);
    msgrcv(f, &q, 100, 1, 0);

    printf("%s", q.mtext);

    return 0;
}
```

## 信号量
### API介绍
```c
#include <sys/types.h>
#include <sys/sem.h>

struct semid_ds {
    struct ipc_perm sem_perm;           /* operation permission struct */
    time_t sem_otime;                   /* last semop() time */
    unsigned long __sem_otime_high;
    time_t sem_ctime;                   /* last time changed by semctl() */
    unsigned long __sem_ctime_high;
    unsigned long sem_nsems;            /* number of semaphores in set */
    unsigned long __glibc_reserved3;
    unsigned long __glibc_reserved4;
};

struct sembuf {
    unsigned short int sem_num;	/* semaphore number */
    short int sem_op;		    /* semaphore operation */
    short int sem_flg;		    /* operation flag */
};


// 创建一个新信号量集或获取一个既有集合的标识符
// nsems: 信号量的数量.如果是创建，必须 >0；如果是获取，必须不大于集合的大小
// 出错时返回-1
int semget (key_t key, int nsems, int semflg);


// 在一个信号量集或集合中的单个信号量上执行各种控制操作
// semnum: 对于单个信号量操作，标志了具体的信号量，对于其他操作则忽略它
// cmd: IPC_RMID，删除信号量及semid_ds
//      IPC_STAT，在arg.buf指向的缓冲器中放置一份与这个信号量集相关联的semid_ds数据结构的副本
//      IPC_SET，在arg.buf指向的缓冲器中放置一份与这个信号量集相关联的semid_ds数据结构的副本
//      GETVAL，返回由semid指定的信号量集中第semnum个信号量的值，无需arg
//      SETVAL，将由semid指定的信号量集中第semnum个信号量的值初始化为arg.val
//      GETALL，获取由semid指向的信号量集中所有信号量的值并将它们放在arg.array指向的数组中。
//      SETALL，使用arg.array指向的数组中的值初始化semid指向的集合中的所有信号量。忽略semnum
//      GETPID，返回上一个在该信号量上执行semop()的进程 ID；称为sempid值。如果没有进程执行过semop()，那么就返回0
//      GETNCNT，返回当前等待该信号量的值增长的进程数；这个值被称为semncnt值
//      GETZCNT，返回当前等待该信号量的值变成0的进程数；这个值被称为semzcnt值
// 出错时返回-1
int semctl (int semid, int semnum, int cmd, ... /* arg */)；


// 在semid标识的信号量集中的信号量上执行一个或多个操作
// sops->sem_op: > 0，加到信号量值上
//               = 0，如果等于0，那么操作将立即结束，否则semop()就会阻塞直到信号量值变成0为止
//               < 0，信号量值减去它
// 出错时返回-1
int semop (int semid, struct sembuf *sops, size_t nsops);
```

### 示例
1. 基本使用
```c
#include <sys/types.h>
#include <sys/sem.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(void) {
    int pid;
    int semid = semget(IPC_PRIVATE, 10, S_IRUSR | S_IWUSR | IPC_CREAT);
    if (semid < 0) {
        printf("get error\n");
        exit(1);
    }

    pid = fork();
    if (pid == 0) {
        printf("Always First\n");
        struct sembuf b;
        b.sem_num = 1;
        b.sem_op = 1;
        semop(semid, &b, 1);
        exit(0);
    } else {
        struct sembuf b;
        b.sem_num = 1;
        b.sem_op = -1;
        semop(semid, &b, 1);
        printf("Always Second\n");
    }

    return 0;
}
```

## 共享内存
### API介绍
```c
#include <sys/types.h>
#include <sys/shm.h>


// 创建一个新的共享内存段，或打开一个既有的共享内存段
// size: 内核以页面大小为单位对齐，若打开一个既有的段，size <= 既有段的大小，但不会产生任何效果
// shmflg: SHM_HUGETLB
//         SHM_NORESERVE
// 出错时返回-1
int shmget (key_t key, size_t size, int shmflg);


// 将 shmid 标识的共享内存段附加到调用进程的虚拟地址空间中
// shmaddr: == NULL，内存被设置到合适的位置（推荐做法，具有更好的可移植性）
//          != NULL且没有设置SHM_RND
//          != NULL且设置了SHM_RND
// 出错时返回(void *)-1
void *shmat (int shmid, const void *shmaddr, int shmflg);


int shmdt (const void *shmaddr);


int shmctl (int shmid, int cmd, struct shmid_ds *buf);
```

### 示例
1. 基本使用
```c
#include <sys/types.h>
#include <sys/shm.h>
#include <sys/sem.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main(void) {
    int pid;
    int semid = semget(IPC_PRIVATE, 10, S_IRUSR | S_IWUSR | IPC_CREAT);
    if (semid < 0) {
        printf("semget error\n");
        exit(1);
    }
    int shmid = shmget(IPC_PRIVATE, 1024, S_IRUSR | S_IWUSR | IPC_CREAT);
    if (shmid < 0) {
        printf("shmget error\n");
        exit(1);
    }

    char *shmp = shmat(shmid, NULL, 0);

    pid = fork();
    if (pid == 0) {
        struct sembuf b;
        b.sem_num = 1;
        b.sem_op = 1;
        memcpy(shmp, "hello, world\n", 14);
        semop(semid, &b, 1);
    } else {
        struct sembuf b;
        b.sem_num = 1;
        b.sem_op = -1;
        semop(semid, &b, 1);
        char buf[128];
        memcpy(buf, shmp, 14);
        printf("%s", buf);
    }

    return 0;
}
```