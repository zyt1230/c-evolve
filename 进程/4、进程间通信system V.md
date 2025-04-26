# 一、了解System V和对象
+ <font style="color:rgb(51,51,51);">对于内核提供的三种通信方式，对于</font>**<font style="color:#DF2A3F;">管道</font>**<font style="color:rgb(51,51,51);">而言，只能实现</font>**<font style="color:#DF2A3F;">单向的数据通信</font>**<font style="color:rgb(51,51,51);">，对于</font><font style="color:#DF2A3F;">信号</font><font style="color:rgb(51,51,51);">通信而言，只能完成</font><font style="color:#DF2A3F;">多进程之间消息的通知</font><font style="color:rgb(51,51,51);">，</font><font style="color:#DF2A3F;">不能起到数据传输的效</font><font style="color:rgb(51,51,51);">果。为了解决上述问题，引入的系统 V进程间通 信。 </font>
+ <font style="color:rgb(51,51,51);">system V提供的进程间通信方式分别是： 消息队列、共享内存、信号量（信号灯集）。</font>
+ <font style="color:rgb(51,51,51);">有关</font>**<font style="color:#DF2A3F;">system V进程间通信对象</font>**<font style="color:rgb(51,51,51);">相关的指令。</font>

> <font style="color:rgb(51,51,51);">ipcs 可以查看所有的信息（消息队列、共享内存、信号量） </font>
>
> <font style="color:rgb(51,51,51);">ipcs -q:可以查看</font>**<font style="color:rgb(51,51,51);">消息队列</font>**<font style="color:rgb(51,51,51);">的信息 </font>
>
> <font style="color:rgb(51,51,51);">ipcs -m:可以查看</font>**<font style="color:rgb(51,51,51);">共享内存</font>**<font style="color:rgb(51,51,51);">的信息 </font>
>
> <font style="color:rgb(51,51,51);">ipcs -s:可以查看</font>**<font style="color:rgb(51,51,51);">信号量</font>**<font style="color:rgb(51,51,51);">的信息 </font>
>
> <font style="color:rgb(51,51,51);">ipcrm -q/m/s ID :可以删除指定ID的IPC对象</font>
>

+ <font style="color:rgb(51,51,51);">上述的三种通信方式，也是</font>**<font style="color:#DF2A3F;">借助内核空间完成的相关通信</font>**<font style="color:rgb(51,51,51);">，</font>**<font style="color:#DF2A3F;">原理是</font>**<font style="color:rgb(51,51,51);">在内核空间创建出相关的对象容器，在进行进程间通信时，可以将信息放入对象中，另一个进程就可以从该容器中取数据了。 </font>
+ <font style="color:rgb(51,51,51);">与内核提供的管道、信号通信不同：</font><font style="color:#DF2A3F;">system V的ipc对象实现了数据传递的</font>**<font style="color:#DF2A3F;">容器与程序相分离</font>**<font style="color:rgb(51,51,51);">，也 就是说，</font>**<font style="color:#DF2A3F;">即使程序以己经结束，但是放入到容器中的数据依然存在，除非将容器手动删除 </font>**<font style="color:rgb(51,51,51);">。                               </font>

# <font style="color:rgb(51,51,51);">二、消息队列</font>
## 1、实现原理
![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1744211576392-b7b8792f-d77f-4816-a766-1d33b9bb6e4c.png)

+ A创建一个信封【里面有信息的类型和正文】，这样信息就封装起来了。
+ ABC都可以往里面放数据，也可以取任意位置数据。
+ 如果要取数据，需要在共同的队列和一样的钥匙打开队列，获得数据。

## 2、需要的函数
```bash
1、创建key值
       #include <sys/types.h>
       #include <sys/ipc.h>
       key_t ftok(const char *pathname, int proj_id); //ftok("/", 'k');
       功能：通过给定的文件以及给定的一个随机值，创建出一个4字节整数的key值，用于system
       V IPC对象的创建
       参数1：一个文件路径，要求是已经存在的文件路径，提供了key值3字节的内容，其中，
       文件的设备号占1字节，文件的inode号占2字节
       参数2：一个随机整数，取后8位（1字节）跟前面的文件共同组成key值，必须是非0的数字
       返回值：成功返回key值，失败返回-1并置位错误码


    | 设备号 (1字节) | inode 号 (2字节) | proj_id (1字节) |

设备号是设备编号
inode号是文件在系统中的唯一标识
如果只有文件路径，没有projid就会导致被破解。


2、通过key值，创建消息队列
       #include <sys/types.h>
       #include <sys/ipc.h>
       #include <sys/msg.h>
       
       int msgget(key_t key, int msgflg);
       功能：通过给定的key值，创建出一个消息队列的对象，并返回消息队列的句柄ID，后期可
       以通过该ID操作整个消息队列
       参数1：key值，该值可以是IPC_PRIVATE,也可以是ftok创建出来的，前者只用于亲缘进程
       间的通信
       参数2：创建标识
       IPC_CREAT:创建并打开一个消息队列，如果消息队列已经存在，则直接打开，亲缘间交流
       IPC_EXCL:确保本次创建处理的是一个新的消息队列，如果消息队列已经存在，则报错，错
误码位EEXIST
       0664：该消息队列的操作权限
       eg: IPC_CREAT|0664  或者  IPC_CREAT|IPC_EXCL|0664
       返回值：成功返回消息队列的ID号，失败返回-1并置位错误码
3、向消息队列中存放数据
       #include <sys/types.h>
       #include <sys/ipc.h>
       #include <sys/msg.h>
       
       int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);
       功能：向消息队列中存放一个指定格式的消息
       参数1：打开的消息队列的id号
       参数2：要发送的消息的起始地址，消息一般定义为一个结构体类型，由用户手动定义
        struct msgbuf {
               long mtype;       /* message type, must be > 0 */   消息的类型
               char mtext[1];    /* message data */    消息正文
               。。。
           };
           （long是8字节，可知前8字节必须是类型）
        参数3：消息正文的大小
        参数4：是否阻塞的标识
       0：标识阻塞形式向消息队列中存放消息，如果消息队列满了，就在该函数处阻塞
       IPC_NOWAIT:标识非阻塞的形式向消息队列中存放消息，如果消息队列满了，直接返回
        返回值：成功返回0，失败返回-1并置位错误码
       
       
4、从消息队列中读取消息

       ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp,int 
msgflg);
       功能：从消息队列中取数指定类型的消息放入给定的容器中
        参数1：打开的消息队列的id号
       参数2：要接收的消息的起始地址，消息一般定义为一个结构体类型，由用户手动定义
        struct msgbuf {
               long mtype;       /* message type, must be > 0 */   消息的类型
               char mtext[1];    /* message data */    消息正文
               。。。
           };
        参数3：消息正文的大小
        参数4：要接收的消息类型
       0:表示每次都取消息队列中的第一个消息，无论类型
       >0:读取队列中第一个类型为msgtyp的消息
       <0:读取队列中的一个消息，消息为绝对值小于msgtyp的第一个消息
       eg：    10-->8-->3-->6-->5-->20-->2
       -5: 会从队列中绝对值小于5的类型的消息中选取第一个消息，就是3   
        参数5：是否阻塞的标识
       0：标识阻塞形式向消息队列中读取消息，如果消息队列空了，就在该函数处阻塞
       IPC_NOWAIT:标识非阻塞的形式向消息队列中读取消息，如果消息队列空了，直接返回
       返回值：成功返回实际读取的正文大小，失败返回-1并置位错误码
       
5、销毁消息队列
       #include <sys/types.h>
       #include <sys/ipc.h>
       #include <sys/msg.h>
       int msgctl(int msqid, int cmd, struct msqid_ds *buf);
       功能：对给定的消息队列执行相关的操作，该操作由cmd参数而定
       参数1：消息队列的ID号
       参数2：要执行的操作
       IPC_RMID：删除一个消息队列，当cmd为该值时，第三个参数可以省略填NULL即可
       IPC_STAT：表示获取当前消息队列的属性，此时第三个参数就是存放获取的消息队列属性
的容器起始地址
       IPC_SET：设置当前消息队列的属性，此时第三个参数就是要设置消息队列的属性数据的起
始地址
       参数3：消息队列数据容器结构体，如果第二个参数为IPC_RMID,则该参数忽略填NULL即可，如果
是 IPC_STAT、 IPC_SET填如下结构体：
       struct msqid_ds {
               struct ipc_perm msg_perm;     /* Ownership and permissions */   
消息队列的拥有者和权限
               time_t          msg_stime;    /* Time of last msgsnd(2) */   最后
一次发送消息的时间
               time_t          msg_rtime;    /* Time of last msgrcv(2) */   最后
一次接收消息的时间
               time_t          msg_ctime;    /* Time of last change */      最后
一次状态改变的时间
               unsigned long   __msg_cbytes; /* Current number of bytes in queue 
(nonstandard) */ 已用字节数
               msgqnum_t       msg_qnum;     /* Current number of messages in 
queue */ 消息队列中消息个数
               msglen_t        msg_qbytes;   /* Maximum number of bytes allowed 
in queue */最大消息个数
               pid_t           msg_lspid;    /* PID of last msgsnd(2) */    最后
一次发送消息的进程pid
               pid_t           msg_lrpid;    /* PID of last msgrcv(2) */    最后
一次读取消息的进程pid
           };
       该结构体的第一个成员类型如下：
       struct ipc_perm {
               key_t          __key;       /* Key supplied to msgget(2) */   key
值
               uid_t          uid;         /* Effective UID of owner */   当前进
程的uid
               gid_t          gid;         /* Effective GID of owner */   当前进
程的组ID
               uid_t          cuid;        /* Effective UID of creator */ 消息队
列创建者的用户id
               gid_t          cgid;        /* Effective GID of creator */ 消息队
列创建者的组id
               unsigned short mode;        /* Permissions */              消息队
列的权限
               unsigned short __seq;       /* Sequence number */          队列号
           };
 返回值：成功返回0，失败返回-1并置位错误码
```

+ 注意事项
    - 对于消息而言，由2部分组成：消息的类型和正文，消息结构体由用户自定义。
    - 对于消息队列而言，任意一个进程都可以向消息队列中发送消息，也可以从消息队列中读取消息。
    - 多个进程，使用相同的key值打开的是同一个消息队列
    - 对消息队列中的消息读取操作也是一次性的，被读取后，消息队列中不存在该消息了。
    - <font style="color:#DF2A3F;">消息队列大小</font>是<font style="color:#DF2A3F;">16k</font>



## 3、实现过程【#x是16进制】
```cpp
#include<sys/types.h>
#include<sys/ipc.h>
#include<sys/msg.h>
#include<myhead.h>
//组建一个消息
struct msgBuf{
    long mtype;
    char mtext[1024];
};
#define MSGSZ (sizeof(struct msgBuf)-sizeof(long)) //正文的大小
int main(int argc,const char* argv[])
{
    //1、创建key值，用于创建出一个消息队列
    key_t key = ftok("/",'k');
    //参数1：已经存在的路径
    //参数2：是一个路径
    if(key==-1){
        perror("ftok error");
        
        return -1;
    }
    printf("key = %#x\n",key);//#x是16进制
    //2、通过key值创建一个消息队列，并返回该消息队列的id
    int msqid = -1;
    if((msqid=msgget(key,IPC_CREAT|0664))==-1){
        perror("msgget error");
        return -1;
    }
    printf("msqid = %d\n",msqid);
    //3、向消息队列中存放消息
    //用户自定义消息
    struct msgBuf buf;
    while(1){
        printf("请输入消息的类型:");
        scanf("%ld",&buf.mtype);
        getchar();//吸收回车符，否则无法输入stdin的内容
        printf("请输入消息正文:");
        fgets(buf.mtext,MSGSZ,stdin);//从终端输入,可以换成FILE*fp
        buf.mtext[strlen(buf.mtext)-1]='\0';//将换行更换成'\0'
        //将上述组装的消息放入消息队列中
        msgsnd(msqid,&buf,MSGSZ,0);//0:阻塞的方式放入消息队列
        printf("消息存入成功\n");
        //判断退出条件
        if(strcmp(buf.mtext,"quit")==0){
            break;
        }
    }
    return 0;
}
```

```cpp
#include<sys/types.h>
#include<sys/ipc.h>
#include<sys/msg.h>
#include<myhead.h>
//组建一个消息
struct msgBuf{
    long mtype;
    char mtext[1024];
};
#define MSGSZ (sizeof(struct msgBuf)-sizeof(long)) //正文的大小
int main(int argc,const char* argv[])
{
    //1、创建key值，用于创建出一个消息队列
    key_t key = ftok("/",'k');
    //参数1：已经存在的路径
    //参数2：是一个路径
    if(key==-1){
        perror("ftok error");
        
        return -1;
    }
    printf("key = %#x\n",key);//#x是16进制
    //2、通过key值创建一个消息队列，并返回该消息队列的id
    int msqid = -1;
    if((msqid=msgget(key,IPC_CREAT|0664))==-1){
        perror("msgget error");
        return -1;
    }
    printf("msqid = %d\n",msqid);
    //3、向消息队列中取消息
    //组建一个消息
    struct msgBuf buf;
    while(1){
        //清空容器
        bzero(&buf,sizeof(buf));
        //读取消息
        msgrcv(msqid,&buf,MSGSZ,3,0);
        //参数4：表示读取的消息类型
        //参数5：表示是否阻塞读取
        printf("读取到的消息为：%s\n",buf.mtext);
        //判断退出条件
        if(strcmp(buf.mtext,"quit")==0){
            break;
        }
    }
    //4、删除消息队列
    if(msgctl(msqid,IPC_RMID,NULL)==-1){
        perror("msgctl error");
        return -1;
    }
    return 0;
}
```

```cpp
删除的方式：ipcrm -q msqid
```

# 二、共享内存
## 1、原理图
![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1745540559634-19a3201e-e9b3-450c-9521-584f3c47443f.png)

+ 在虚拟地址外的物理地址，创建一个很小的物理内存，然后进程a和进程b都可以使用这个物理内存的物理地址，进行操作。

> 注意：
>
> **<font style="color:#DF2A3F;">如果A进程对这个物理内存进行修改了，B进程操作的是一块新的物理内存了</font>**。
>

## 2、共享内存API
```cpp
1、创建key值
       #include <sys/types.h>
       #include <sys/ipc.h>
       key_t ftok(const char *pathname, int proj_id); //ftok("/", 'k');
       功能：通过给定的文件以及给定的一个随机值，创建出一个4字节整数的key值，用于system V 
IPC对象的创建
       参数1：一个文件路径，要求是已经存在的文件路径，提供了key值3字节的内容，其中，
           文件的设备号占1字节，文件的inode号占2字节
       参数2：一个随机整数，取后8位（1字节）跟前面的文件共同组成key值，必须是非0的数字
       返回值：成功返回key值，失败返回-1并置位错误码
      
2、通过key值创建共享内存段
 #include <sys/ipc.h>
       #include <sys/shm.h>
       int shmget(key_t key, size_t size, int shmflg);
       功能：申请指定大小的物理内存，映射到内核空间，创建出共享内存段
       参数1：key值，可以是IPC_PRIVATE,也可以是ftok创建出来的key值
       参数2：申请的大小，是一页（4096字节）的整数倍，并且向上取整
       参数3：创建标识
       IPC_CREAT:创建并打开一个共享内存，如果共享内存已经存在，则直接打开
       IPC_EXCL:确保本次创建处理的是一个新的共享内存，如果共享内存已经存在，则报错，错
误码位EEXIST
       0664：该共享内存的操作权限
       eg: IPC_CREAT|0664  或者  IPC_CREAT|IPC_EXCL|0664
       返回值：成功返回共享内存段的id，失败返回-1并置位错误码
       
3、将共享内存段的地址映射到用户空间
       #include <sys/types.h>
       #include <sys/shm.h>
       void *shmat(int shmid, const void *shmaddr, int shmflg);
       功能：将共享内存段映射到用户空间

       参数1：共享内存的id号
       参数2：物理内存的起始地址，一般填NULL，由系统自动选择一个合适的对齐页
       参数3：对共享内存段的操作
       0：表示读写操作
       SHM_RDONLY:只读
       返回值：成功返回用于操作共享内存的指针，失败返回(void*)-1并置位错误码
4、释放共享内存的映射关系
       int shmdt(const void *shmaddr);
       功能：将进程与共享内存的映射取消
       参数：共享内存的指针
       返回值：成功返回0，失败返回-1并置位错误码
       
5、共享内存的控制函数
       #include <sys/ipc.h>
       #include <sys/shm.h>
       int shmctl(int shmid, int cmd, struct shmid_ds *buf);
       功能：根据给定的不同的cmd执行不同的操作
       参数1：共享内存的ID
       参数2：要操作的指令
       IPC_RMID:删除共享内存段，第三个参数可以省略
       IPC_STAT:获取当前共享内存的属性
       IPC_SET:设置当前共享内存的属性
       参数3：如果参数2为IPC_RMID，则参数3可以省略填NULL，如果参数2为另外两个，
           参数3填如下结构体变量
       struct shmid_ds {
               struct ipc_perm shm_perm;    /* Ownership and permissions */
               size_t          shm_segsz;   /* Size of segment (bytes) */
               time_t          shm_atime;   /* Last attach time */
               time_t          shm_dtime;   /* Last detach time */
               time_t          shm_ctime;   /* Last change time */
               pid_t           shm_cpid;    /* PID of creator */
               pid_t           shm_lpid;    /* PID of last shmat(2)/shmdt(2) */
               shmatt_t        shm_nattch;  /* No. of current attaches */
               ...
           };
          该结构体的第一个成员结构体：
          struct ipc_perm {
               key_t          __key;    /* Key supplied to shmget(2) */
               uid_t          uid;      /* Effective UID of owner */
               gid_t          gid;      /* Effective GID of owner */
               uid_t          cuid;     /* Effective UID of creator */
               gid_t          cgid;     /* Effective GID of creator */
               unsigned short mode;     /* Permissions + SHM_DEST and
                                           SHM_LOCKED flags */
               unsigned short __seq;    /* Sequence number */
           };
        返回值：成功返回0，失败饭hi-1并置位错误码
```

+ 注意
    - 4096是向上取整数
    - ftok的参数2 是 低8位
    - shmat物理内存的起始地址，一般填NULL，由系统自动选择一个合适的对齐页
    - ![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1745560274128-e652f397-43aa-4a31-bb64-255f3e8d557f.png)



## 2、实现过程
```cpp
#include <myhead.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include<sem.h>
#define PAGE_SIZE 4096 //一页的大小
int main(int argc, const char *argv[]) {
  // 1、创建key值
  key_t key = ftok("/", 'k');
  if (key == -1) {
    perror("ftok error");
    return -1;
  }
  printf("key = %#x\n", key);
  // 2、通过key值创建共享内存段
  int shmid = -1;
  if ((shmid = shmget(key, PAGE_SIZE, IPC_CREAT | 0664)) == -1) {
    perror("shmget error");
    return -1;
  }
  printf("shmid = %d \n", shmid);
  // 3、将共享内存段映射到用户空间
  char *addr = (char *)shmat(shmid, NULL, 0);
  // NULL表示让系统自动寻址对齐页
  // 0表示对共享内存段读写操作
  if (addr == (void *)-1) {
    perror("shmat error");
    return -1;
  }
  printf("addr = %p\n", addr); //输出共享内存段映射的地址
  // 4、对共享内存操作
  //char wbuf[128] = "";
  while (1) {
    //11、申请0号信号灯的资源

    printf("请输入>>>");
    fgets(addr, sizeof(addr), stdin);
    addr[strlen(addr)-1]='\0';
    if (strcmp(addr, "quit") == 0) {
      break;
    }
  }
  // 5、取消映射
  if ((shmdt(addr)) == -1) {
    perror("取消映射\n");
    return -1;
  }
  // 6. 删除共享内存（由写入端负责删除）
  if (shmctl(shmid, IPC_RMID, NULL) == -1) {
    perror("shmctl error");
    return -1;
  }
  return 0;
}

```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1745553341414-9e38155f-891f-413d-a33e-c74e3a06e7a7.png)

```cpp
#include<myhead.h>
#include<time.h>
#include<sys/shm.h>
#include<sys/types.h>
#include<sys/ipc.h>
#include<unistd.h>
#define PAGE_SIZE 4096 //一页的大小
int main(int argc,const char* argv[])
{
    //1、创建key值
    key_t key = ftok("/",'k');
    if(key==-1){
        perror("ftok error");
        return -1;
    }
    printf("key = %#x\n",key);
    //2、通过key值创建共享内存段
    int shmid = -1;
    if((shmid=shmget(key,PAGE_SIZE,IPC_CREAT|0664))==-1){
        perror("shmget error");
        return -1;
    }
    printf("shmid = %d \n",shmid);
    //3、将共享内存段映射到用户空间
    char* addr = (char*)shmat(shmid,NULL,0);
    //NULL表示让系统自动寻址对齐页
    //0表示对共享内存段读写操作
    if(addr==(void*)-1){
        perror("shmat error");
        return -1;
    }
    printf("addr = %p\n",addr);//输出共享内存段映射的地址
    //4、对共享内存操作
    char wbuf[128]="";
    while(1){
        sleep(2);
        printf("读取到的消息为：%s\n",addr);
        
        if(strcmp(addr,"quit")==0){
            break;
        }
    }
    //5、取消映射
    if (shmdt(addr) == -1) {
        perror("shmdt error");
        return -1;
    }

    return 0;
}
```

> 注意：
>
> + 共享内存VS消息队列：
>     - 共享内存：维持了消息的<font style="color:#DF2A3F;">时效性</font>。
>     - 消息队列：维持了消息的<font style="color:#DF2A3F;">不丢失性</font>。
> + 消息队列：可能存在<font style="color:#DF2A3F;">竞态</font>，需要与信号量一起使用。
> + 使用共享内存跟使用指针一样，使用时，<font style="color:#DF2A3F;">无需再继续用户空间和内核空间的切换</font>了，所以说，共享内存是所有进程间通信方式中效率最高的一种方式。
>

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1745569740075-e4af4a1c-8e8e-4158-b3a6-3facc09f8a36.png)

# 三、信号量【解决谁先执行，谁后执行】
## 1、了解原理图
![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1745569394388-43def6fe-8f7b-4c35-a9a1-c1e18e99cf14.png)

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1745569410898-8538ab6c-5f8c-4698-9b34-1e4f8c09a2b0.png)

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1745569427934-b1c7ae93-02d8-462a-8f1d-af1a2e748344.png)

## 2、信号量API【P（申请资源）V（释放资源）】 


```cpp
1、创建key值
       #include <sys/types.h>
       #include <sys/ipc.h>
       key_t ftok(const char *pathname, int proj_id); //ftok("/", 'k');
       功能：通过给定的文件以及给定的一个随机值，创建出一个4字节整数的key值，用于system V 
IPC对象的创建
       参数1：一个文件路径，要求是已经存在的文件路径，提供了key值3字节的内容，其中，文件的设备
号占1字节，文件的inode号占2字节
       参数2：一个随机整数，取后8位（1字节）跟前面的文件共同组成key值，必须是非0的数字
       返回值：成功返回key值，失败返回-1并置位错误码
       
2、通过key值创建信号量集
       #include <sys/types.h>
       #include <sys/ipc.h>
       #include <sys/sem.h>
       int semget(key_t key, int nsems, int semflg);
       功能：通过给定的key值创建一个信号量集
       参数1：：key值，该值可以是IPC_PRIVATE,也可以是ftok创建出来的，前者只用于亲缘进程间的
通信
       参数2：信号量数组中信号量的个数
       参数3：创建标识
       IPC_CREAT:创建并打开一个信号量集，如果信号量集已经存在，则直接打开
       IPC_EXCL:确保本次创建处理的是一个新的信号量集，如果信号量集已经存在，则报错，错
误码位EEXIST
       0664：该信号量集的操作权限
       eg: IPC_CREAT|0664  或者  IPC_CREAT|IPC_EXCL|0664
       返回值：成功返回信号量集的id，失败返回-1并置位错误码
       
       
3、关于信号量集的操作：P（申请资源）V（释放资源）   
       #include <sys/types.h>
       #include <sys/ipc.h>
       #include <sys/sem.h>
       int semop(int semid, struct sembuf *sops, size_t nsops);
       功能：完成对信号量数组的操作
       参数1：信号量数据ID号
       参数2：有关信号量操作的结构体变量起始地址，该结构体中包含了操作的信号量编号和申请还是释
放的操作
       struct sembuf
       {
         unsigned short sem_num;  /* semaphore number */    要操作的信号量的编号
           short          sem_op;   /* semaphore operation */  要进行的操作，大于0
表示释放资源，小于0表示申请资源
           short          sem_flg;  /* operation flags */   操作标识位，0标识阻塞方
式，IPC_NOWAIT表示非阻塞
       }
       参数3：本次操作的信号量的个数
       返回值：成功返回0，失败返回-1并置位错误码
       
4、关于信号量集的控制函数
       #include <sys/types.h>
       #include <sys/ipc.h>
       #include <sys/sem.h>
       int semctl(int semid, int semnum, int cmd, ...);
       功能：执行有关信号量集的控制函数，具体控制内容取决于cmd
       参数1：信号量集的ID
       参数2：要操作的信号量的编号，编号是从0开始
       参数3：要执行的操作
       IPC_RMID:表示删除信号量集，cmd为该值时，参数2可以忽略，参数4可以不填
       SETVAL：表示对参数2对应的信号量进行设置操作（初始值）
       GETVAL：表示对参数2对应的信号量进行获取值操作
       SETALL：设置信号量集中所有信号量的值
       GETALL:获取信号量集中的所有信号量的值
       IPC_STAT:表示获取当前信号量集的属性
       IPC_SET:表示设置当前信号量集的属性、
       参数4：根据不同的cmd值，填写不同的参数值，所以该处是一个共用体变量
       union semun {
               int              val;    /* Value for SETVAL */      设置信号量的值
               struct semid_ds *buf;    /* Buffer for IPC_STAT, IPC_SET */   关于
信号量集属性的操作
               unsigned short  *array;  /* Array for GETALL, SETALL */    对于信
号量集中所有信号量的操作
               struct seminfo  *__buf;  /* Buffer for IPC_INFO
                                           (Linux-specific) */
           };
        返回值：成功时：SETVAL、IPC_RMID返回0，GETVAL返回当前信号量的值，失败返回-1并置位
错误码
        
        例如：
        1) 给0号信号量设置初始值为1
        union semun us;           //定义一个共用体变量
        us.val = 1;               //对该共用体变量赋值
        semctl(semid, 0, SETVAL, us);    //该函数就完成了对0号信号量设置初始值为1的操作
        
        2) 删除信号量集
        semctl(semid, 0, IPC_RMID);

```

+ **<font style="color:#DF2A3F;">将上述函数进行2次封装，封装成为只有信号量集的创建、申请资源、释放资源、销毁信号量集</font>**。



## 3、实现过程
### 3.1、二次的封装
```cpp
#ifndef _SEM_H
#define _SEM_H
//创建信号量集并初始化
int create_sem(int semcount);//创建semcount个信号灯
//申请资源操作
int P(int semid,int semno);//semid是信号灯集的id ,semno是哪一个信号灯
//释放资源操作semno表示要释放资源的信号量编号
int V(int semid,int semno);
//删除信号灯集
int delete_sem(int semid);

#endif 
```

```cpp
#include<myhead.h>
#include<sys/types.h>
#include<sys/sem.h>
#include<errno.h>
#include"sem.h"
union semun{
    int val;//设置信号量的值
    struct semid_ds* buf;//关于信号量集属性的操作
    unsigned short* array;//对应信号量集中所有的信号量操作
    struct seminfo*_buf;//Buffer for IPC_INFO
};
//定义一个关于信号量初始化函数
static int init_sem(int semid,int semno){
    int val = -1;
    printf("亲输入第%d个信号量的初始值：",semno+1);//让用户输入信号量的初始值
    scanf("%d",&val);
    getchar();//吸收回车
    //调用semctl完成设置
    union semun us;
    us.val=val;//设置初始值
    if(semctl(semid,semno,SETVAL,us)==-1){
        perror("semctl (SETVAL) error");
        return -1;
    }
    return 0;
}
//创建信号量集并初始化
int create_sem(int semcount){//创建semcount个信号灯
    //1、创建key值
    key_t key = ftok("/",'k');
    if(key==-1){
        perror("ftok erro");
        return -1;
    }
    //2、创建信号灯集
    int semid = -1;
    if((semid=semget(key,semcount,IPC_CREAT|IPC_EXCL|0664))==-1){
        if(errno==EEXIST){//表示信号量集已经存在，直接打开即可
            semid = semget(key,semcount,IPC_CREAT|0664);
            return semid;
        }
        perror("semget error");
        return -1;
    }
    //3、循环将信号量集的所有信号量进行初始化
    //该操作，只有在第一次创建信号量集时需要进行操作，后面再打开该信号量
    //集时，就无需进行初始化操作。
    for(int i =0;i<semcount;i++){
        init_sem(semid,i);////调用自定义函数将每个信号量进行初始化
    }
    return 0;
}
//申请资源操作
int P(int semid,int semno){
    //定义一个结构体变量
    struct sembuf buf;
    buf.sem_num=semno;//要操作的信号编号
    buf.sem_op=-1;//-1表示要申请该信号量的资源
    buf.sem_flg=0;//表示阻塞
    //调用semop函数
    if(semop(semid,&buf,1)==-1){//操作1个灯
        perror("P error");
        return -1;
    }
    return 0;

}//semid是信号灯集的id ,semno是哪一个信号灯
//释放资源操作semno表示要释放资源的信号量编号
int V(int semid,int semno){
    //定义一个结构体变量
    struct sembuf buf;
    buf.sem_num=semno;//要操作的信号编号
    buf.sem_op=1;//1表示要释放该信号量的资源
    buf.sem_flg=0;//表示阻塞
    //调用semop函数
    if(semop(semid,&buf,1)==-1){//操作1个灯
        perror("V error");
        return -1;
    }
    return 0;

}
//删除信号灯集
int delete_sem(int semid){
    //调用semctl函数完成对该信号量集的删除操作
    if(semctl(semid,0,IPC_RMID)==-1){
        perror("delete error");
        return -1; 
    }
    return 0;
}


```

```cpp
#include"sem.h"
#include<myhead.h>
#include<sys/shm.h>
#include<sys/types.h>
#include<sys/ipc.h>
#include<unistd.h>

#define PAGE_SIZE 4096 //一页的大小
int main(int argc,const char* argv[])
{
    //00、调用自定义函数：创建并打开信号量集------------------------------------------
    int semid =create_sem(2);//这里有2个进程，所以是2
    //1、创建key值
    key_t key = ftok("/",'k');
    if(key==-1){
        perror("ftok error");
        return -1;
    }
    printf("key = %#x\n",key);
    //2、通过key值创建共享内存段
    int shmid = -1;
    if((shmid=shmget(key,PAGE_SIZE,IPC_CREAT|0664))==-1){
        perror("shmget error");
        return -1;
    }
    printf("shmid = %d \n",shmid);
    //3、将共享内存段映射到用户空间
    char* addr = (char*)shmat(shmid,NULL,0);
    //NULL表示让系统自动寻址对齐页
    //0表示对共享内存段读写操作
    if(addr==(void*)-1){
        perror("shmat error");
        return -1;
    }
    printf("addr = %p\n",addr);//输出共享内存段映射的地址
    //4、对共享内存操作
    
    while(1){
        //11、申请1号信号灯的资源--------------------------------
        P(semid,1);
        printf("读取到的消息为：%s\n",addr);
        //22、释放号信号灯的资源---------------------------------
        if(strcmp(addr,"quit")==0){
            break;
        }
        V(semid,0);
    }
    //5、取消映射
    if (shmdt(addr) == -1) {
        perror("shmdt error");
        return -1;
    }
    // 6. 删除共享内存
    if (shmctl(shmid, IPC_RMID, NULL) == -1) {
        perror("shmctl error");
        return -1;
    }
    //44、删除信号量集-----------------------------------------
    //delete_sem(semid);//这里不用再删了。
    return 0;
}
```



```cpp
#include <myhead.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include"sem.h"
#define PAGE_SIZE 4096 //一页的大小
int main(int argc, const char *argv[]) {
  //00、创建并打开一个信号量集------------------------------------------
  int semid =create_sem(2);//这里有2个进程，所以是2


  // 1、创建key值
  key_t key = ftok("/", 'k');
  if (key == -1) {
    perror("ftok error");
    return -1;
  }
  printf("key = %#x\n", key);
  // 2、通过key值创建共享内存段
  int shmid = -1;
  if ((shmid = shmget(key, PAGE_SIZE, IPC_CREAT | 0664)) == -1) {
    perror("shmget error");
    return -1;
  }
  printf("shmid = %d \n", shmid);
  // 3、将共享内存段映射到用户空间
  char *addr = (char *)shmat(shmid, NULL, 0);
  // NULL表示让系统自动寻址对齐页
  // 0表示对共享内存段读写操作
  if (addr == (void *)-1) {
    perror("shmat error");
    return -1;
  }
  printf("addr = %p\n", addr); //输出共享内存段映射的地址
  // 4、对共享内存操作
  //char wbuf[128] = "";
  while (1) {
    //11、申请0号信号灯的资源--------------------------------
    P(semid,0);
    printf("请输入>>>");
    fgets(addr, sizeof(addr), stdin);
    addr[strlen(addr)-1]=0;
    //22、释放1号信号灯的资源---------------------------------
    V(semid,1);
    if (strcmp(addr, "quit") == 0) {
      break;
    }
  }
  // 5、取消映射
  if ((shmdt(addr)) == -1) {
    perror("取消映射\n");
    return -1;
  }
  //44、删除信号量集-----------------------------------------
  delete_sem(semid);
  
  return 0;
}

```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1745631021054-56bba603-e6e3-4a28-bf15-89f26efd4de3.png)



### 


