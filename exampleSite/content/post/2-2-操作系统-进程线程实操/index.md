+++
author = "coucou"
title = "操作系统——进程&线程实操"
date = "2023-08-01"
description = "操作系统专题之进程&线程实操篇"
categories = [
    "操作系统"
]
tags = [
    "操作系统","进程&线程实操"
]
+++
![](1.jpg)

## Linux 多进程和多线程编程

### 多进程

```c
n=fork() 
// 创建一个子进程，该命令执行后将产生一个子进程，子进程的PCB以及进程空间都是由父进程拷贝而来的，因而两者相同，差别是返回值。
对子进程，n＝0；
对父进程，n等于子进程的进程号（即n>0）；
n=-1，创建失败。
    
// 通过判断返回值便可安排父进程和子进程的后继活动，使两者完成不同的任务，通常用法：
n=fork();
if (n==0)
	{/*子进程所要执行的代码*/ }
else
	{/*父进程所要执行的代码*/}
```

>**exec( )系列**
>系统调用 exec()系列，也可**用于新程序的运行**。fork()只是将父进程的用户级上下文拷贝到新进程中，而 exec()系列可以将一个可执行的二进制文件覆盖在新进程的用户级上下文的存储空间上，以更改新进程的用户级上下文。exec()系列中的系统调用都完成相同的功能，它们把一个新程序装入内存，来改变调用进程的执行代码，从而形成新进程。
>
>如果exec( )调用成功，调用进程将被覆盖，然后从新程序的入口开始执行，这样就产生了一个新进程，新进程的进程标识符id 与调用进程相同。 exec( )没有建立一个与调用进程并发的子进程，而是用新进程取代了原来进程。**所以 exec()调用成功后，没有任何数据返回，这与 fork()不同**。exec()系列系统调用在UNIX系统库unistd.h中，共有execl、execlp、execle、execv、execvp五个，其基本功能相同，只是以不同的方式来给出参数。

```c
// 一种是直接给出参数的指针，如：
Int execl(char *path, char *arg0[,arg1,...argn],0);
// 另一种是给出指向参数表的指针，如：
Int execv(char *path, char *argv[]);

// exec( )和fork( )联合使用
// 系统调用 exec 和 fork()联合使用能为程序开发提供有力支持。用 fork()建立子进程，然后在子进程中使用exec( )，这样就实现了父进进程与一个与它完全不同子进程的并发执行。一般，wait、exec 联合使用的模型为：
int status;
  ............
if (fork( )= =0)
  {
  ...........;
execl(...);
  ...........;
  }
wait(&status);

// wait(NULL)的作用是:使子进程优先执行
```

### 管道

>**管道**
>
>指能够连接一个**写进程**和一个**读进程**的、并允许它们以生产者—消费者方式进行通信的一个共享文件，又称为pipe文件。由写进程从管道的写入端（句柄1）将数据写入管道，而读进程则从管道的读出端（句柄0）读出数据。
>**管道的类型**
>
>1、有名管道
>一个可以在文件系统中**长期存在的、具有路径名**的文件。用系统调用**mknod( )**建立。它克服无名管道使用上的局限性，可让更多的进程也能利用管道进行通信。因而其它进程可以知道它的存在，并能利用路径名来访问该文件。对有名管道的访问方式与访问其他文件一样，需先用open( )打开。
>2、无名管道
>一个**临时文件**。利用**pipe( )**建立起来的无名文件（无路径名）。只用该系统调用所返回的文件描述符来标识该文件，故只有调用pipe( )的进程及其子孙进程才能识别此文件描述符，才能利用该文件（管道）进行通信。
>3、pipe文件的建立
>分配磁盘和内存索引结点、为读进程分配文件表项、为写进程分配文件表项、分配用户文件描述符
>4、读/写进程互斥
>为使读、写进程互斥地访问pipe文件，需使**各进程互斥地访问**pipe文件索引结点中的直接地址项。因此，每次进程在访问pipe文件前，都需检查该索引结点是否已被上锁。若是，进程便睡眠等待，否则，将其上锁，进行读/写。操作结束后解锁，并唤醒因该索引结点上锁而睡眠的进程。

```c
1、pipe()
// 功能：建立一无名管道
// 参数定义
int  pipe(filedes);
int  filedes[2];
// 其中，filedes[1]是写入端，filedes[0]是读出端。
// 该函数使用头文件如下：
#include <unistd.h>
#inlcude <signal.h>
#include <stdio.h>

2、read()
//功能：从fd所指示的文件中读出nbyte个字节的数据，并将它们送至由指针buf所指示的缓冲区中。如该文件被加锁，等待，直到锁打开为止。
//参数定义
int  read(fd,buf,nbyte);
int  fd;
char *buf;
unsigned  nbyte;

3、write()
//系统调用格式
// 功能：把nbyte 个字节的数据，从buf所指向的缓冲区写到由fd所指向的文件中。如文件加锁，暂停写入，直至开锁。
write (fd,buf,nbyte)
    
4、exit()
// 终止进程的执行。
// 系统调用格式：
void exit(status)
int status;
//其中，status是返回给父进程的一个整数。
UNIX/LINUX利用exit( )来实现进程的自我终止，通常父进程在创建子进程时，应在进程的末尾安排一条exit( )，使子进程自我终止。exit(0)表示进程正常终止，exit(1)表示进程运行有错，异常终止。
如果调用进程在执行exit( )时，其父进程正在等待它的终止，则父进程可立即得到其返回的整数。核心须为exit( )完成以下操作：
（1）关闭软中断
（2）回收资源
（3）写记帐信息
（4）置进程为“僵死状态”
    
5、wait()
//等待子进程运行结束。如果子进程没有完成，父进程一直等待。wait( )将调用进程挂起，直至其子进程因暂停或终止而发来软中断信号为止。如果在wait( )前已有子进程暂停或终止，则调用进程做适当处理后便返回。
//系统调用格式：
int  wait(status)　
int  *status;
其中，status是用户空间的地址。它的低8位反应子进程状态，为0表示子进程正常结束，非0则表示出现了各种各样的问题；高8位则带回了exit( )的返回值。exit( )返回值由系统给出。

6、lockf（files，function,size）
// 用做锁定文件的某些段或者整个文件，本函数适用的头文件为：
#include<unistd.h>
参数定义：int lockf(files,function,size)
         Int files,function;
         Long size;
// 其中，：files是文件描述符；function是锁定和解锁；1表示锁定，0表示解锁。Size是锁定或解锁的字节数，若为0，表示从文件的当前位置到文件尾。
```

### 共享存储区

>共享存储区（Share Memory）是UNIX系统中通信速度最高的一种通信机制。该机制可使若干进程共享主存中的某一个区域，且使该区域出现（映射）在多个进程的虚地址空间中。另一方面，一个进程的虚地址空间中又可连接多个共享存储区，每个共享存储区都有自己的名字。当进程间欲利用共享存储区进行通信时，必须先在主存中建立一共享存储区，然后将它附接到自己的虚地址空间上。此后，进程对该区的访问操作，与对其虚地址空间的其它部分的操作完全相同。进程之间便可通过对共享存储区中数据的读、写来进行直接通信。图示列出二个进程通过共享一个共享存储区来进行通信的例子

![img](share.jpg)

>应当指出，共享存储区机制只为进程提供了用于实现通信的共享存储区和对共享存储区进行操作的手段，然而并未提供对该区进行互斥访问及进程同步的措施。因而当用户需要使用该机制时，**必须自己设置同步和互斥措施才能保证实现正确的通信**。

```c
1、shmget( )
// 创建、获得一个共享存储区。
系统调用格式：
                 shmid=shmget(key,size,flag)
// 该函数使用头文件如下：
#include<sys/types.h>
#include<sys/ipc.h>
#include<sys/shm.h>
int  shmget(key,size,flag);
key_t  key;
int  size,flag;
// 其中，key是共享存储区的名字；size是其大小（以字节计）；flag是用户设置的标志，如IPC_CREAT。IPC_CREAT表示若系统中尚无指名的共享存储区，则由核心建立一个共享存储区；若系统中已有共享存储区，便忽略IPC_CREAT。
附：
        操作允许权                  八进制数
         用户可读                     00400
         用户可写                     00200
         小组可读                     00040
         小组可写                     00020 
         其它可读                     00004
         其它可写                     00002
         
		控制命令                    值
		IPC_CREAT                0001000
		IPC_EXCL                 0002000
例：shmid=shmget(key,size,(IPC_CREAT|0400))
// 创建一个关键字为key，长度为size的共享存储区
    
2、shmat( )
//共享存储区的附接。从逻辑上将一个共享存储区附接到进程的虚拟地址空间上。
系统调用格式：
                 virtaddr=shmat(shmid,addr,flag)
该函数使用头文件如下：
#include<sys/types.h>
#include<sys/ipc.h>
#include<sys/shm.h>

char  *shmat(shmid,addr,flag);
int  shmid,flag;
char  * addr; 
// 其中，shmid是共享存储区的标识符；addr是用户给定的，将共享存储区附接到进程的虚地址空间；flag规定共享存储区的读、写权限，以及系统是否应对用户规定的地址做舍入操作。其值为SHM_RDONLY时，表示只能读；其值为0时，表示可读、可写；其值为SHM_RND（取整）时，表示操作系统在必要时舍去这个地址。该系统调用的返回值是共享存储区所附接到的进程虚地址viraddr。

3、shmdt()
// 把一个共享存储区从指定进程的虚地址空间断开。
系统调用格式：
             shmdt(addr)
该函数使用头文件如下：
#include<sys/types.h>
#include<sys/ipc.h>
#include<sys/shm.h>
    
int  shmdt(addr);
char  addr;
// 其中，addr是要断开连接的虚地址，亦即以前由连接的系统调用shmat( )所返回的虚地址。调用成功时，返回0值，调用不成功，返回-1。

4、shmctl()
// 共享存储区的控制，对其状态信息进行读取和修改。
系统调用格式：
               shmctl(shmid,cmd,buf)
该函数使用头文件如下：
#include<sys/types.h>
#include<sys/ipc.h>
#include<sys/shm.h>
    
int  shmctl(shmid,cmd,buf);
int  shmid,cmd;
struct  shmid_ds  *buf;
// 其中，buf是用户缓冲区地址，cmd是操作命令。命令可分为多种类型：
（1）用于查询有关共享存储区的情况。如其长度、当前连接的进程数、共享区的创建者标识符等；
（2）用于设置或改变共享存储区的属性。如共享存储区的许可权、当前连接的进程计数等；
（3）对共享存储区的加锁和解锁命令；
（4）删除共享存储区标识符等。
// 上述的查询是将shmid所指示的数据结构中的有关成员，放入所指示的缓冲区中；而设置是用由buf所指示的缓冲区内容来设置由shmid所指示的数据结构中的相应成员。
```

### 消息的创建,发送和接收

>**消息**
>
>消息（message）是一个格式化的可变长的信息单元。消息机制允许由一个进程给其它任意的进程发送一个消息。当一个进程收到多个消息时，可将它们排成一个消息队列。消息使用二种重要的数据结构：一是消息首部，其中记录了一些与消息有关的信息，如消息数据的字节数；二个消息队列头表，其每一表项是作为一个消息队列的消息头，记录了消息队列的有关信息。
>
>1、消息机制的数据结构
>
>（1）消息首部
>
>记录一些与消息有关的信息，如消息的类型、大小、指向消息数据区的指针、消息队列的链接指针等。
>
>（2）消息队列头表
>
>其每一项作为一个消息队列的消息头，记录了消息队列的有关信息如指向消息队列中第一个消息和指向最后一个消息的指针、队列中消息的数目、队列中消息数据的总字节数、队列所允许消息数据的最大字节总数，还有最近一次执行发送操作的进程标识符和时间、最近一次执行接收操作的进程标识符和时间等。
>
>2、消息队列的描述符
>
>UNIX中，每一个消息队列都有一个称为关键字（key）的名字，是由用户指定的；消息队列有一消息队列描述符，其作用与用户文件描述符一样，也是为了方便用户和系统对消息队列的访问。

```c
1. msgget()
//创建一个消息，获得一个消息的描述符。核心将搜索消息队列头表，确定是否有指定名字的消息队列。若无，核心将分配一新的消息队列头，并对它进行初始化，然后给用户返回一个消息队列描述符，否则它只是检查消息队列的许可权便返回。
系统调用格式：
	msgqid=msgget(key,flag)
该函数使用头文件如下：
#include<sys/types.h>
#include<sys/ipc.h>
#include<sys/msg.h>

int msgget(key,flag)
key_t  key;
int  flag;
// key是用户指定的消息队列的名字；flag是用户设置的标志和访问方式。如  IPC_CREAT |0400       是否该队列已被创建。无则创建，是则打开；
IPC_EXCL |0400        是否该队列的创建应是互斥的。
msgqid 是该系统调用返回的描述符，失败则返回-1。
    
2. msgsnd()
// 发送一消息。向指定的消息队列发送一个消息，并将该消息链接到该消息队列的尾部。
系统调用格式：
             msgsnd(msgqid,msgp,size,flag)
该函数使用头文件如下：
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

int msgsnd(msgqid,msgp,size,flag)
int msgqid,size,flag;
struct msgbuf  * msgp;
其中msgqid是返回消息队列的描述符；msgp是指向用户消息缓冲区的一个结构体指针。缓冲区中包括消息类型和消息正文，即
{
    long  mtype;             /*消息类型*/
    char  mtext[ ];           /*消息的文本*/
}
// size指示由msgp指向的数据结构中字符数组的长度；即消息的长度。这个数组的最大值由MSG-MAX( )系统可调用参数来确定。flag规定当核心用尽内部缓冲空间时应执行的动作:进程是等待，还是立即返回。若在标志flag中未设置IPC_NOWAIT位，则当该消息队列中的字节数超过最大值时，或系统范围的消息数超过某一最大值时，调用msgsnd进程睡眠。若是设置IPC_NOWAIT，则在此情况下，msgsnd立即返回。
对于msgsnd( )，核心须完成以下工作：
（1）对消息队列的描述符和许可权及消息长度等进行检查。若合法才继续执行，否则返回；
（2）核心为消息分配消息数据区。将用户消息缓冲区中的消息正文，拷贝到消息数据区；
（3）分配消息首部，并将它链入消息队列的末尾。在消息首部中须填写消息类型、消息大小和指向消息数据区的指针等数据；
（4）修改消息队列头中的数据，如队列中的消息数、字节总数等。最后，唤醒等待消息的进程。
    
3. msgrcv()
// 接受一消息。从指定的消息队列中接收指定类型的消息。
系统调用格式：
           msgrcv(msgqid,msgp,size,type,flag)
本函数使用的头文件如下：
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

int  msgrcv(msgqid,msgp,size,type,flag)
int  msgqid,size,flag;
struct  msgbuf  *msgp;
long  type;
// 其中，msgqid,msgp,size,flag与msgsnd中的对应参数相似，type是规定要读的消息类型，flag规定倘若该队列无消息，核心应做的操作。如此时设置了IPC_NOWAIT标志，则立即返回，若在flag中设置了MS_NOERROR，且所接收的消息大于size，则核心截断所接收的消息。
对于msgrcv系统调用，核心须完成下述工作：
（1）对消息队列的描述符和许可权等进行检查。若合法，就往下执行；否则返回；
（2）根据type的不同分成三种情况处理：
	type=0，接收该队列的第一个消息，并将它返回给调用者；
	type为正整数，接收类型type的第一个消息；
	type为负整数，接收小于等于type绝对值的最低类型的第一个消息。
（3）当所返回消息大小等于或小于用户的请求时，核心便将消息正文拷贝到用户区，并从消息队列中删除此消息，然后唤醒睡眠的发送进程。但如果消息长度比用户要求的大时，则做出错返回。
    
4. msgctl()
// 消息队列的操纵。读取消息队列的状态信息并进行修改，如查询消息队列描述符、修改它的许可权及删除该队列等。
系统调用格式：
             msgctl(msgqid,cmd,buf);
本函数使用的头文件如下：
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

int msgctl(msgqid,cmd,buf);
int  msgqid,cmd;
struct  msgqid_ds  *buf;
// 其中，函数调用成功时返回0，不成功则返回-1。buf是用户缓冲区地址，供用户存放控制参数和查询结果；cmd是规定的命令。命令可分三类：
（1）IPC_STAT。查询有关消息队列情况的命令。如查询队列中的消息数目、队列中的最大字节数、最后一个发送消息的进程标识符、发送时间等；
（2）IPC_SET。按buf指向的结构中的值，设置和改变有关消息队列属性的命令。如改变消息队列的用户标识符、消息队列的许可权等；
（3）IPC_RMID。消除消息队列的标识符。
// msgqid_ds  结构定义如下：
struct  msgqid_ds
   {  struct  ipc_perm  msg_perm;     /*许可权结构*/
      short  pad1[7];                 /*由系统使用*/
      ushort msg_qnum;               /*队列上消息数*/
      ushort msg_qbytes;              /*队列上最大字节数*/
      ushort msg_lspid;               /*最后发送消息的PID*/
      ushort msg_lrpid;               /*最后接收消息的PID*/
      time_t msg_stime;               /*最后发送消息的时间*/
      time_t msg_rtime;               /*最后接收消息的时间*/
      time_t msg_ctime;               /*最后更改时间*/
     };
struct  ipc_perm
   {  ushort uid;                     /*当前用户*/
      ushort gid;                     /*当前进程组*/
      ushort cuid;                    /*创建用户*/
      ushort cgid;                    /*创建进程组*/
      ushort mode;                   /*存取许可权*/
      { short pid1; long pad2;}         /*由系统使用*/  
    }
```

### 线程与同步

>POSIX线程（POSIX threads），简称Pthreads，是线程的POSIX标准。 该标准定义了创建和操纵线程的一整套API。在类Unix操作系统（Unix、Linux、Mac OS X等）中，都使用Pthreads作为操作系统的线程。

```c
#include <pthread.h> // 头文件

数据类型：
pthread_t：线程ID

操纵函数
1、int pthread_create(pthread_t* thread, pthread_attr_t* attr, void* (*start_routine)(void*), void* arg);
//创建一个由调用线程控制的新的线程并发运行。新的线程使用start_routine作为实现体，并以arg作为第一个参数。

//新的线程可以通过调用pthread_exit显式结束，或者通过start_routine return来隐式结束。 其中后者等价于调用pthread_exit并以start_routine 的返回值作为退出码。

//attr参数指明新线程的属性，如果attr=NULL,则使用默认属性：新线程是joinable(not detached),和默认的调度策略(非实时)
    
//返回值：如果成功，新线程的指针会被存储到thread的参数中，并返回0。如果错误则一个非0的错误码返回。 如果返回EAGAIN，没有足够的系统资源创建一个线程，或者已经存在大于PTHREAD_THREADS_MAX个活跃线程。

2、int pthread_join(pthread_t th, void **thread_return);      // 阻塞当前的线程，直到另外一个线程运行结束
// 挂载一个在执行的线程直到线程通过调用pthread_exit或者cancelled结束。

//如果thread_return不为空，则线程th的返回值会保存到thread_return所指的区域。
    
//th的返回值是它给pthread_exit的参数，或者是pthread_canceled 如果是被cancelled的。
    
//被依附的线程th必须是joinable状态。一定不能是detached通过使用pthread_detach或者pthread_create中使用pthread_create_detached属性。
    
//当一个joinable线程结束时，他的资源(线程描述符和堆栈)不会被释放直到另一个线程对它执行pthread_join操作。
    
如果成功，返回值存储在thread_return中，并返回0，否则返回错误码：
•	ESRCH:找不到指定线程
•	EINVAL:线程th是detached或者已经存在另一个线程在等待线程th结束
•	EDEADLK:th的参数引用它自己(即线程不能join自身)


互斥锁类型：
pthread_mutex_t

操纵函数
pthread_mutex_init()：    // 初始化互斥锁
pthread_mutex_lock()：    // 加锁
pthread_mutex_unlock()：  // 解锁

gcc main.c -lpthread   // 使用 Pthreads 时，编译时必须加上 -lpthread
```

### 信号量

```c
#include <semaphore.h>  // 头文件

1. sem_init()
int sem_init(sem_t *sem, int pshared, unsigned int value);
// sem: 要初始化的信号量
// pshared: 非0表示此信号量是在进程间共享，0表示是线程间共享
// value： 是信号量的初始值。
    
2. sem_wait()
int sem_wait(sem_t *sem);     // 等待信号量，如果信号量的值大于0,将信号量的值减1,立即返回。如果信号量的值为0,则线程阻塞。
    
3. sem_post()
int sem_post(sem_t *sem);    // 释放信号量，让信号量的值加1。
```

### 文件操作

```c
1. fopen()     // 打开一个文件。
    
FILE * fopen(const char * filename, const char * mode);
// mode模式说明:
    r   以只读方式打开文件，该文件必须存在。
    w   打开只写文件，若文件存在则文件长度清为0，即该文件内容会消失。若文件不存在则建立该文件.
    +   以可读写方式打开文件，该文件必须存在。
    b   打开一个二进制文件，只允许读写数据。
    a   以附加的方式打开只写文件。若文件不存在，则会建立该文件，如果文件存在，写入的数据会被加到文件尾，即文件原先的内容会被保留。（EOF符保留）
        
2. fclose()     // 关闭一个文件。如果成功关闭，fclose 返回 0，否则返回EOF（-1）。

int fclose(FILE *stream);

3. fgetc()      // 从文件中读取字符。返回读取的一个字节。如果读到文件末尾返回EOF。

int fgetc(FILE *stream);

4. fputc()       // 输出一个字符到文件中。

int fputc(int ch, FILE *stream);

5. putchar()     // 输出一个字符到标准输出（即显示器）中。等价于：fputc(ch, STDOUT);

int putchar(int ch);
```

