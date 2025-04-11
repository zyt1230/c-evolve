# 一、进程间通信方式和引入原因
+ <font style="color:rgb(51,51,51);">由于</font>**<font style="color:#DF2A3F;">多个进程的用户空间是相互独立的</font>**<font style="color:rgb(51,51,51);">，其栈区、堆区、静态区的数据都是彼此</font>**<font style="color:#DF2A3F;">私有</font>**<font style="color:rgb(51,51,51);">的，所以</font><font style="color:#DF2A3F;">不可 能通过用户空间中的区域完成多个进程之间数据的通信</font><font style="color:rgb(51,51,51);"> </font>
+ <font style="color:#DF2A3F;">可以使用</font>**<font style="color:#DF2A3F;">外部文件</font>**<font style="color:#DF2A3F;">来完成</font>**<font style="color:#DF2A3F;">多个进程之间数据的传递</font>**<font style="color:rgb(51,51,51);">，一个进程向文件中写入数据，另一个进程从文 件中读取数据。</font><font style="color:#DF2A3F;">该方式要必须保证</font>**<font style="color:#DF2A3F;">写进程先执行</font>**<font style="color:#DF2A3F;">，然后</font>**<font style="color:#DF2A3F;">再执行读进程</font>**<font style="color:#DF2A3F;">，要保证进程执行的</font>**<font style="color:#DF2A3F;">同步性</font>**<font style="color:rgb(51,51,51);">  。【一般不用这个】</font>
+ <font style="color:rgb(51,51,51);">我们可以</font>**<font style="color:#DF2A3F;">利用内核空间来完成对数据的通信工作</font>**<font style="color:rgb(51,51,51);">，</font>**<font style="color:#DF2A3F;">本质上</font>**<font style="color:rgb(51,51,51);">，在内核空间创建</font>**<font style="color:rgb(51,51,51);">一个特殊的区域</font>**<font style="color:rgb(51,51,51);">，</font>**<font style="color:#DF2A3F;">一个 进程</font>**<font style="color:rgb(51,51,51);">向该区域中</font>**<font style="color:rgb(51,51,51);">存放数据</font>**<font style="color:rgb(51,51,51);">，</font>**<font style="color:#DF2A3F;">另一个进程</font>**<font style="color:rgb(51,51,51);">可以从该区域中</font>**<font style="color:rgb(51,51,51);">读取数据</font>**<font style="color:rgb(51,51,51);"> 。</font>

```cpp
#include<unistd.h>
#include<sys/types.h>
#include<wait.h>
#include<myhead.h>
//int num = 520;//定义一个变量
int main(int argc,const char* argv[])
{
    int num = 520;//定义一个变量
    pid_t pid = fork();//创建子进程
    if(pid>0){
        //父进程
        num=1314;
        printf("父进程中num = %d \n",num);
        wait(NULL);//等待回收子进程
    }else if(pid ==0){
        //子进程
        sleep(3);
        printf("子进程中 = %d\n",num);
     //子进程num在全局还是局部都是520        
    }else{
        strerror(errno);
        return -1;
    }
    return 0;
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1744103195786-046fa56d-b9fd-45a4-bf07-d978a9caba89.png)

+ **<font style="color:#DF2A3F;">引入原因</font>**<font style="color:rgb(51,51,51);">：用户空间中的数据，不能作为多个进程之间数据交换的容器 。</font>

# <font style="color:rgb(51,51,51);">二、进程间通信的基础概念</font>
## 1、基础知识点
+ <font style="color:rgb(51,51,51);">IPC :interprocess communication 进程间通信</font>
+ <font style="color:rgb(51,51,51);">使用</font>**<font style="color:#DF2A3F;">内核空间</font>**<font style="color:rgb(51,51,51);">来完成多个进程间相互通信，根据使用的容器或方式不同，分为三类通信机制。</font>
+ <font style="color:rgb(51,51,51);">进程间通信方式分类：</font>

```cpp
1、内核提供的通信方式（传统的通信方式）【你发消息，我不能发】
 无名管道
 有名管道
 信号
2、system V提供的通信方式【你发消息，我也发消息】
 消息队列
 共享内存
 信号量（信号灯集）
3、套接字通信：socket 网络通信（跨主机通信）【互相发消息的基础上，跨主机通信】


- 前2种只能一个主机上的多个进程进行。

```

## 2、内核提供
### 2.1、管道的原理和文件类型
+ **<font style="color:#DF2A3F;">管道是特殊的文件</font>**：它不用于存储数据等等，**<font style="color:#DF2A3F;">只用于进程间通信</font>**。
+ <font style="color:rgb(51,51,51);">在内核空间创建出一个管道通信，一个进程可以将数据</font>**<font style="color:rgb(51,51,51);">写入管道</font>**<font style="color:rgb(51,51,51);">，经由管道缓冲到另一个进程中</font>**<font style="color:rgb(51,51,51);">读取数据</font>**<font style="color:rgb(51,51,51);">。 </font>

```cpp
文件类型
文件类型：bcd-lsp
b:块设备文件
c：字符设备文件
d：目录文件，文件夹
-：普通文件
l：链接文件
s：套接字文件（网络编程）
p：管道文件
```



### 2.2、无名管道
+ <font style="color:rgb(51,51,51);">无名管道：顾名思义就是</font>**<font style="color:rgb(51,51,51);">没有名字</font>**<font style="color:rgb(51,51,51);">的管道，会在</font>**<font style="color:rgb(51,51,51);">内存中创建</font>**<font style="color:rgb(51,51,51);">出该管道，</font>**<font style="color:rgb(51,51,51);">不存在于文件系统</font>**<font style="color:rgb(51,51,51);">，随着 </font>**<font style="color:rgb(51,51,51);">进程结束而消失</font>**<font style="color:rgb(51,51,51);"> 。</font>
+ <font style="color:rgb(51,51,51);">无名管道</font>**<font style="color:#DF2A3F;">仅适用于亲缘进程间通信</font>**<font style="color:rgb(51,51,51);">，</font>**<font style="color:#DF2A3F;">不适用于非亲缘进程间通信</font>**<font style="color:rgb(51,51,51);"> 。</font>

```cpp
       #include <unistd.h>
       int pipe(int fildes[2]);
       功能：创建一个无名管道，并返回该管道的两个文件描述符
       参数：是一个整型数组，用于返回打开的管道的两端的文件描述符， fildes[0]表示读端 
fildes[1]表示写端
       返回值：成功返回0，失败返回-1并置位错误码

```

+ **<font style="color:#DF2A3F;">pipe要放在 子进程创建前，否则子进程的pipe是新创建的</font>**。

```cpp
#include<myhead.h>
#include<unistd.h>
#include<sys/types.h>
#include<wait.h>
int main(int argc,const char* argv[])
{
    //可以在此创建管道文件，
    //并返回该管道文件的两端，
    //那么父子进程都会拥有该管道两端的文件描述符
    int fildes[2];//存放管道文件的2端文件描述符

    //创建无名管道，并返回该管道的2端文件描述符
    int res =pipe(fildes);
    if(res==-1){
        perror("pipe error");
        return -1;
    }
    printf("fildes[0]=%d ,fildes[1]=%d\n",fildes[0],fildes[1]);//打印3 4，因为012最小未分配原则

    pid_t pid = fork();
    
    //也不可以在此创建管道文件
    //因为如果在此进程，那么父进程和子进程分别创建一个无名管道

    if(pid>0){
        //父进程
        char wbuf[138]="hello world";
        //如果在此处创建管道，父进程能使用，但是子进程不能使用
        //因为子进程没有管道文件的读端和写端文件描述符

        //将数据发给子进程，用写的方式
        write(fildes[1],wbuf,sizeof(wbuf));
        close(fildes[1]);
        wait(NULL);
    }else if(pid==0){
        //子进程
        //
        char rbuf[138]="";
        read(fildes[0],rbuf,sizeof(rbuf));

        printf("收到父进程的数据为%s \n",rbuf);
        close(fildes[0]);
        exit(EXIT_SUCCESS);
    }else{
        perror("fork error");
        return -1;
    }
    return 0;
}
```

> **<font style="color:#DF2A3F;">管道通信的特点</font>**：
>
> <font style="color:rgb(51,51,51);">1、管道可以实现</font>**<font style="color:rgb(51,51,51);">自己给自己发消息 </font>**
>
> ![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1744128970151-084f76c0-54bc-4a2c-afea-b44ff25f52c0.png)
>
> <font style="color:rgb(51,51,51);">2、对管道中数据的</font>**<font style="color:rgb(51,51,51);">操作是一次性的</font>**<font style="color:rgb(51,51,51);">，当管道中的数据被读取后，就从管道中消失了，再读取时会被阻塞 </font>
>
> <font style="color:rgb(51,51,51);">3、管道文件的</font>**<font style="color:rgb(51,51,51);">大小：64K [65536/1024=64K]</font>**
>
> <font style="color:rgb(51,51,51);">4、由于</font>**<font style="color:#DF2A3F;">返回的是管道文件的文件描述符</font>**<font style="color:rgb(51,51,51);">，所以对管道的操作</font>**<font style="color:rgb(51,51,51);">只能是文件IO相关函数</font>**<font style="color:rgb(51,51,51);">，但是，</font>**<font style="color:rgb(51,51,51);">不可以使用 lseek对光标进行偏移</font>**<font style="color:rgb(51,51,51);">，</font>**<font style="color:#DF2A3F;">必须做到先进先出</font>**<font style="color:rgb(51,51,51);">  。</font>
>
> **<font style="color:rgb(51,51,51);">5、</font>****<font style="color:rgb(64, 64, 64);"> 当读端存在时</font>**
>
> + **<font style="color:rgb(64, 64, 64);">写端行为</font>**<font style="color:rgb(64, 64, 64);">：</font>
>     - <font style="color:rgb(64, 64, 64);">可以一直写入数据，</font>**<font style="color:rgb(64, 64, 64);">直到管道缓冲区满</font>**<font style="color:rgb(64, 64, 64);">（通常为 64KB）。</font>
>     - <font style="color:rgb(64, 64, 64);">缓冲区满后，</font>`<font style="color:rgb(64, 64, 64);">write()</font>`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">会</font>**<font style="color:rgb(64, 64, 64);">阻塞</font>**<font style="color:rgb(64, 64, 64);">，直到读端读取部分数据腾出空间。</font>
> + **<font style="color:rgb(64, 64, 64);">读端行为</font>**<font style="color:rgb(64, 64, 64);">：</font>
>     - <font style="color:rgb(64, 64, 64);">可以一直读取数据，</font>**<font style="color:rgb(64, 64, 64);">直到管道为空</font>**<font style="color:rgb(64, 64, 64);">。</font>
>     - <font style="color:rgb(64, 64, 64);">如果管道为空，</font>`<font style="color:rgb(64, 64, 64);">read()</font>`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">会</font>**<font style="color:rgb(64, 64, 64);">阻塞</font>**<font style="color:rgb(64, 64, 64);">，直到写端写入新数据。</font>
>
> #### **<font style="color:rgb(64, 64, 64);">6、 当读端不存在时</font>**
> + **<font style="color:rgb(64, 64, 64);">写端行为</font>**<font style="color:rgb(64, 64, 64);">：</font>
>     - <font style="color:rgb(64, 64, 64);">继续写入数据会触发</font><font style="color:rgb(64, 64, 64);"> </font>**<font style="color:rgb(64, 64, 64);">管道破裂（Broken Pipe）</font>**<font style="color:rgb(64, 64, 64);">。</font>
>     - <font style="color:rgb(64, 64, 64);">内核向写端进程发送</font><font style="color:rgb(64, 64, 64);"> </font>`<font style="color:rgb(64, 64, 64);">SIGPIPE</font>`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">信号（默认行为是终止进程）。</font>
>     - <font style="color:rgb(64, 64, 64);">如果进程捕获或忽略 </font>`<font style="color:rgb(64, 64, 64);">SIGPIPE</font>`<font style="color:rgb(64, 64, 64);">，</font>`<font style="color:rgb(64, 64, 64);">write()</font>`<font style="color:rgb(64, 64, 64);"> 会返回 </font>`<font style="color:rgb(64, 64, 64);">-1</font>`<font style="color:rgb(64, 64, 64);">，并设置 </font>`<font style="color:rgb(64, 64, 64);">errno</font>`<font style="color:rgb(64, 64, 64);"> 为 </font>`<font style="color:rgb(64, 64, 64);">EPIPE</font>`<font style="color:rgb(64, 64, 64);">。</font>
>
> #### **<font style="color:rgb(64, 64, 64);">7、 当写端存在时</font>**
> + **<font style="color:rgb(64, 64, 64);">读端行为</font>**<font style="color:rgb(64, 64, 64);">：</font>
>     - <font style="color:rgb(64, 64, 64);">可以读取管道中的数据，</font>**<font style="color:rgb(64, 64, 64);">有多少读多少</font>**<font style="color:rgb(64, 64, 64);">。</font>
>     - <font style="color:rgb(64, 64, 64);">如果管道为空，</font>`<font style="color:rgb(64, 64, 64);">read()</font>`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">会</font>**<font style="color:rgb(64, 64, 64);">阻塞</font>**<font style="color:rgb(64, 64, 64);">，直到写端写入数据。</font>
>
> #### **<font style="color:rgb(64, 64, 64);">8、当写端不存在时</font>**
> + **<font style="color:rgb(64, 64, 64);">读端行为</font>**<font style="color:rgb(64, 64, 64);">：</font>
>     - <font style="color:rgb(64, 64, 64);">读取管道中剩余的数据，</font>**<font style="color:rgb(64, 64, 64);">有多少读多少</font>**<font style="color:rgb(64, 64, 64);">。</font>
>     - <font style="color:rgb(64, 64, 64);">管道为空后，</font>`<font style="color:rgb(64, 64, 64);">read()</font>`<font style="color:rgb(64, 64, 64);"> </font>**<font style="color:rgb(64, 64, 64);">立即返回 0</font>**<font style="color:rgb(64, 64, 64);">（不会阻塞），表示 EOF（文件结束符）。</font>
>
> <font style="color:rgb(51,51,51);">9、</font>**<font style="color:rgb(51,51,51);">管道通信是半双工通信方式</font>**<font style="color:rgb(51,51,51);"> ：</font>
>
> <font style="color:rgb(51,51,51);">单工：只能进程A向B发送消息 </font>
>
> <font style="color:rgb(51,51,51);">半双工：同一时刻只能A向B发消息 </font>
>
> <font style="color:rgb(51,51,51);">全双工：任意时刻，AB可以互相通信</font>
>



```cpp
#include<myhead.h>
#include<unistd.h>
int main(int argc,const char* argv[])
{
    int fildes[2];
    if(pipe(fildes)==-1){
        perror("pipe error");
        return -1;
    }
    printf("fildes[0] = %d fildes[1] = %d\n",fildes[0],fildes[1]);
    //定义2个容器
    char wbuf[128]="ni hao xingqiu";
    char rbuf[128]="";
    //将wbuf中的数据写入管道文件中,从管道中放入rbuf中
    write(fildes[1],wbuf,sizeof(wbuf));
    read(fildes[0],rbuf,sizeof(rbuf));
    printf("rbuf = %s\n",rbuf);
    close(fildes[0]);
    close(fildes[1]);
    return 0;
}
```

```cpp
#include<myhead.h>
#include<unistd.h>
int main(int argc,const char* argv[])
{
    int fildes[2];
    if(pipe(fildes)==-1){
        perror("pipe error");
        return -1;
    }
    printf("fildes[0] = %d fildes[1] = %d\n",fildes[0],fildes[1]);
    //定义2个容器
    char buf= 'A';//定义一个字符变量
    int count =0;//记录管道的大小
    while(1)
    {
        write(fildes[1],&buf,1);
        count++;
        printf("count = %d \n",count);
    }
    
    return 0;
}
```

### 2.3、有名管道---管道文件要第一个创建，后马上进行别的运行。
#### 2.3.1、使用和理解
+ <font style="color:rgb(51,51,51);">顾名思义就是有名字的管道文件，会在文件系统中创建一个真实存在的管道文件</font>
+ <font style="color:rgb(51,51,51);">既可以完成</font>**<font style="color:#DF2A3F;">亲缘进程间通信</font>**<font style="color:rgb(51,51,51);">，也可以完成</font>**<font style="color:#DF2A3F;">非亲缘进程间通信</font>**

```cpp
       #include <sys/types.h>
       #include <sys/stat.h>
       int mkfifo(const char *pathname, mode_t mode);
       功能：创建一个管道文件，并存在与文件系统中
       参数1：管道文件的名称
       参数2：管道文件的权限，内容详见open函数的mode参数
       返回值：成功返回0，失败返回-1并置位错误码
      
     注意：管道文件被创建后，其他进程就可以进行打开读写操作了，但是，必须要保证当前管道文件的
两端都打开后，才能进行读写操作，否则函数会在open处阻塞
           有名管道文件主要是创建管道文件，注意：如果管道文件已经存在，则mkfifo函数会报错。

```

> 运行过程
>
> 必须马上运行管道文件
>
> 然后运行其他文件。./s    ./r
>
> 否则管道文件可能变**普通文件**。
>

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1744160095632-5c12d37b-3e5f-43c0-b953-a9bfdd78b0aa.png)

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1744163346646-758302b0-8157-4b7f-adea-8929fcdedd2b.png)

```cpp
#include<myhead.h>
#include<unistd.h>
#include<sys/stat.h>
#include<sys/types.h>
int main(int argc,const char* argv[])
{
    //该文件主要负责创建管道文件，注意：如果管道文件已经存在，则mkfifo函数会报错
    if(mkfifo("./myfifo",0664)==-1){
        perror("mkfifo error");
        return -1;
    }
    printf("管道创建成功\n");
    
    return 0;
}
```

```cpp
#include<myhead.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<unistd.h>
#include<fcntl.h>
int main(int argc, char const *argv[])
{
    //打开管道文件
    int sfd =-1;
    if((sfd=open("./myfifo",O_WRONLY))==-1){
        perror("open error");
        return -1;
    }
    //准备要写入的数据
    char wbuf[128] = "";
    while(1){
        
        printf(">>>");
        fgets(wbuf,sizeof(wbuf),stdin);//从终端输入
        
        wbuf[strlen(wbuf)-1]=0;//将换行符换成\0
        //将数据写入管道
        int res =write(sfd,wbuf,strlen(wbuf));
        printf("res=%d\n",res);
        if(strcmp(wbuf,"quit")==0){
            break;
        }
    }
    close(sfd);
    return 0;
}

```

```cpp
#include<myhead.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<unistd.h>
#include<fcntl.h>
int main(int argc, char const *argv[])
{
    //打开管道文件
    int rfd =-1;
    if((rfd=open("./myfifo",O_RDONLY))==-1){
        perror("open error");
        return -1;
    }
    
    char rbuf[128] = "";
    while(1){
        //将管道清空
        bzero(rbuf,sizeof(rbuf));
        //从管道文件中读取数据
        int res =read(rfd,rbuf,sizeof(rbuf));
        printf("收到数据为：%s ,res=%d\n",rbuf,res);       
        
        if(strcmp(rbuf,"quit")==0){
            break;
        }
    }
    close(rfd);
    return 0;
}
```

---

#### 2.3.2、练习
<font style="color:rgb(51,51,51);">练习：使用有名管道实现，两个进程之间相互通信，提示：可以使用多进程或多线程 。</font>

+ 一个管道只能半双工通信。但是2各管道就可以全双工通信。
+ 

### 2.4、信号
#### 2.4.1、了解通信原理
```plain
1、信号是软件模拟硬件的中断功能，信号是软件实现的，中断是硬件实现的
 中断：停止当前正在执行的事情，去做另一件事情
2、信号是linux内核实现的，没有内核就没有信号的概念
3、用户可以给进程发信号：例如键入ctrl+c
   内核可以向进程发送信号：例如SIGPIPE
   一个进程可以给另一个进程发送信号，需要通过相关函数来完成
4、信号通信是属于异步通信工作
 同步：表示多个任务 有先后顺序 的执行，例如去银行办理业务。
 异步：表示多个任务 没有先后顺序 执行，例如你在敲代码，你妈妈在做饭。
```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1744180848771-af3ac9b9-1cf2-4ed8-a6f7-29dcdf44c1b5.png)

#### 2.4.2、信号种类和功能
```plain
1、一共可以发射62个信号，前32个是稳定信号，后面是不稳定信号
2、常用的信号
SIGHUP：当进程所在的终端被关闭后，终端会给运行在当前终端的每个进程发送该信号，默认结束进程
SIGINT：中断信号，当用户键入ctrl + c时发射出来
SIGQUIT：退出信号，当用户键入ctrl + /是发送，退出进程
SIGKILL：杀死指定的进程
SIGSEGV：当指针出现越界访问时，会发射，表示段错误
SIGPIPE：当管道破裂时会发送该信号
SIGALRM：当定时器超时后，会发送该信号
SIGSTOP：暂停进程，当用户键入ctrl+z时发射
SIGTSTP：也是暂停进程
SIGTSTP、SIGUSR2 ：留给用户自定义的信号，没有默认操作
SIGCHLD：当子进程退出后，会向父进程发送该信号
3、有两个特殊信号：SIGKILL和SIGSTOP，这两个信号既不能被捕获，也不能被忽略
```

```plain
有两个特殊信号：SIGKILL和SIGSTOP，这两个信号既不能被捕获，也不能被忽略
```

```plain
 #include <signal.h>
       typedef void (*sighandler_t)(int);
       sighandler_t signal(int signum, sighandler_t handler);
       功能：将信号与信号处理方式绑定到一起
       参数1：要处理的信号
       参数2：处理方式
       SIG_IGN：忽略
       SIG_DFL：默认，一般信号的默认操作都是杀死进程
       typedef void (*sighandler_t)(int)：用户自定义的函数
       返回值：成功返回处理方式的起始地址，失败返回SIG_ERR并置位错误码
        
        注意：只要程序与信号绑定一次，后续但凡程序收到该信号，对应的处理方式就会立即响应
```

```cpp
void handler(int signo)
{
    if(signo == SIGINT)
   {
        printf("用户键入了ctrl + c\n");
   }
}

if(signal(SIGINT, handler) == SIG_ERR)
    {
        perror("signal error");
        return -1;
    }

```

```cpp
if(signal(SIGINT, SIG_IGN) == SIG_ERR)
{
    perror("signal error");
    return -1;
}

```

```plain
if(signal(SIGINT, SIG_DFL) == SIG_ERR)
   {
        perror("signal error");
        return -1;
   }

```

#### 2.4.3、使用信号方式完成对僵尸进程的回收
```plain
#include<myhead.h>
#include<unistd.h>
#include<sys/stat.h>
#include<sys/types.h>
#include<signal.h>
#include<wait.h>
void handler(int signo){
    if(signo==SIGCHLD){
        //确保 所有僵尸进程都被回收，防止信号丢失导致部分僵尸残留。
        while(waitpid(-1,NULL,WNOHANG)>0);
    }
}
int main(int argc,const char* argv[])
{
    //当子进程退出后，会向父进程发送一个SIGCHILD的信号，我们可以将其捕获，在信号处理函数中将子进程回收资源
    if(signal(SIGCHLD,handler)==SIG_ERR){
        perror("signal error");
        return -1;
    }
    //10个僵尸进程
    for(int i=0;i<10;i++){
        sleep(2);
        if(fork()==0){//创建10个子进程
            exit(EXIT_SUCCESS);
        }
    }
    while(1);
    return 0;
}
```

#### 2.4.4、信号发送函数kill、raise
```plain
       #include <signal.h>
       int kill(pid_t pid, int sig);
       功能：向指定进程或进程组发送信号
       参数1：进程号或进程组号
       >0:表示向执行进程发送信号
       =0：向当前进程所在的进程组中的所有进程发送信号
       =-1：向所有进程发送信号
       <-1:向指定进程组发送信号，进程组的ID号为给定pid的绝对值
       参数2：要发送的信号
       返回值：成功返回0，失败返回-1并置位错误码
       
       
       #include <signal.h>
       int raise(int sig);
       功能：向自己发送信号   等价于：kill(getpid(), sig);
       参数：要发送的信号
       返回值：成功返回0，失败返回非0数组
```

```cpp
#include<myhead.h>
#include<unistd.h>
#include<sys/stat.h>
#include<sys/types.h>
#include<signal.h>
#include<wait.h>
//定义信号处理函数
void handler(int signo){
    if(signo==SIGUSR1){
        printf("逆子，何至于此\n");
        raise(SIGKILL);//向自己发送自杀信号
        //等价于kill(getpid,SIGKILL)
    }
}
int main(int argc,const char* argv[])
{
    //将子进程发送的信号绑定到指定功能中
    if(signal(SIGUSR1,handler)==SIG_ERR){

    }
    pid_t pid = fork();
    if(pid>0){
        //父进程
        while(1){
            printf("我真的还想活400年\n");
            sleep(1);
        }
    }else if(pid ==0){
        sleep(5);
        printf("红尘已经看破,叫上父亲一起死\n");
        kill(getppid(),SIGUSR1);//SIGUSR1是自定义
        exit(EXIT_SUCCESS);//退出进程
    }else{
        perror("pid error");
        return -1;
    }
    return 0;
}
```

### <font style="color:rgb(51,51,51);">3、总结：信号可以完成多个进程间通知作用，但是，不能进行数据传输功能</font>
