# 一、打开文件：fopen

```c
使用的头文件#include<stdio.h>
FILE* fopen(const char*path,const char*mode)
功能：打开一个文件，并返回当前文件的文件指针
参数1：表示文件路径，是字符串
参数2：打开方式，也是字符串
    r      以只读的形式打开文件，文件光标定位在开头部分
    r+     以读写的形式打开文件，文件光标定位在开头部分
    w      以只写的形式打开文件，如果文件不存在，就创建文件，如果文件存在，就清空，
光标定位在开头
    w+     以读写的形式打开文件，如果文件不存在，就创建文件，如果文件存在，就清空，
光标定位在开头
    a      以追加（结尾写）的形式打开文件，如果文件不存在就创建文件，光标定位在文件末
尾
    a+     以读写的形式打开文件，如果文件不存在就创建文件，如果文件存在，读取操作光
标定位在开头，写操作光标定位在结尾

返回值：成功返回打开的文件指针，失败返回NULL并置位错误码
```

+ 例子

```c
int main(int argc, const char *argv[])
{
    //1、定义一个文件指针
    FILE *fp = NULL;
    //以只读的形式打开一个不存在的文件,并将返回结果存入到fp指针中
    //fp = fopen("./file.txt", "r");    
    //此时会报错，原因是以只读的形式打开一个不存在的文件，是不允许的
    //以只写的形式打开一个不存在的文件,如果文件不存在就创建一个空的文件，如果文件存在就清空
    fp = fopen("./file.txt", "w");           
    if(fp == NULL)
   {
        printf("fopen error\n");
        return -1;
   }
    printf("fopen success\n");
    //2、关闭文件
    fclose(fp);

    return 0;
}
```

# 二、关闭文件：fclose

```cpp
       #include <stdio.h>
       int fclose(FILE *fp);
       功能：关闭指定的文件
       参数：要关闭的文件指针（由fopen返回的结果）
       返回值：成功执行返回0，失败返回EOF并置位错误码

```

# 三、错误码

1> 概念：当内核提供的函数出错后，内核空间会向用户空间反馈一个错误信息，由于错误信息比较多也比较复杂，系统就给每种不同的错误信息起了一个编号，用一个整数表示，这个整数就是错误码

2> 错误码都是大于或等于0的数字：



```cpp
#define EPERM           1     /* Operation not permitted */   操作受限
#define ENOENT           2     /* No such file or directory */ 没有当前文件或目录
#define ESRCH           3     /* No such process */     没有进程
#define EINTR           4     /* Interrupted system call */ 中断系统调用
#define EIO             5     /* I/O error */
#define ENXIO           6     /* No such device or address */
#define E2BIG           7     /* Argument list too long */
#define ENOEXEC         8     /* Exec format error */
#define EBADF           9     /* Bad file number */
#define ECHILD         10     /* No child processes */
#define EAGAIN         11     /* Try again */
#define ENOMEM         12     /* Out of memory */
#define EACCES         13     /* Permission denied */
#define EFAULT         14     /* Bad address */
#define ENOTBLK         15     /* Block device required */
#define EBUSY           16     /* Device or resource busy */
#define EEXIST         17     /* File exists */
#define EXDEV           18     /* Cross-device link */
#define ENODEV         19     /* No such device */
#define ENOTDIR         20     /* Not a directory */
#define EISDIR         21     /* Is a directory */
#define EINVAL         22     /* Invalid argument */
#define ENFILE         23     /* File table overflow */
#define EMFILE         24     /* Too many open files */
#define ENOTTY         25   /* Not a typewriter */
#define ETXTBSY         26     /* Text file busy */
#define EFBIG           27     /* File too large */
#define ENOSPC         28     /* No space left on device */
#define ESPIPE         29     /* Illegal seek */
```

+ 0表示**成功**
+ 2表示**文件或目录不存在**
+ 关于错误码的处理 函数(strerror 、perror)

```cpp
使用man perror找到它的文章

#include <errno.h>   //错误码所在的头文件，里面定义了一个全局变量
int errno;


```

![](file:///C:/Users/LEGION/Pictures/d425f9c5-91d5-4964-a5e7-ac829668887f.png)

+ 打印错误码

```cpp
#include "stdio.h"
#include <errno.h>
int main(int argc,const char* argv[]){
    FILE* fp =NULL;//防止野指针
    fp=fopen("./file.txt","w");
    if(fp==NULL){
        printf("fopen error\n",errno);
        //打印错误信息和错误码
        return -1;
    }
    printf("fopen success\n");
    return 0;
}


```

+ 打印错误码并显示错误码的信息

```cpp

//将错误码对应的错误信息转换处理
       #include <string.h>
       char *strerror(int errnum);
       功能：将错误码转换为错误信息描述
       参数：错误码
       返回值：错误信息字符串描述

//只打印错误码对应的错误信息的函数
       #include <stdio.h>
       void perror(const char *s);
       功能：输出当前错误码对应的错误信息
       参数：提示符号，会原样打印出来，并且会在提示数据后面加上冒号，并输出完后，自动换行
       返回值：无
```

+ 例如

```cpp
#include"stdio.h"        //标准的输入输出头文件
#include<errno.h>          //错误码所在的头文件
#include<string.h>         //字符串处理的头文件
int main(int argc, const char *argv[])
{
    //1、定义一个文件指针
    FILE *fp = NULL;
    fp = fopen("./file.txt", "r");           
    if(fp == NULL)
   {
        //printf("fopen error: %d, errmsg:%s\n", errno, strerror(errno));     //
打印错误信息，并输出错误码 2
        perror("fopen error");      //打印当前错误码对应的错误信息
        return -1;
   }
    printf("fopen success\n");
    //2、关闭文件
    fclose(fp);

    return 0;
}
```

# 四、单字符读写：fputc/fgetc

## 1、ctags的使用

+ 安装ctags

```cpp
sudo yum install ctags #centos
sudo apt install universal-ctags  #ubuntu
```



+ cd /usr/include 执行sudo ctags -R
+ `<font style="color:rgb(64, 64, 64);">-R</font>`<font style="color:rgb(64, 64, 64);"> 表示递归遍历当前目录及其子目录。</font>
+ <font style="color:rgb(64, 64, 64);">执行后会在当前目录生成 </font>`<font style="color:rgb(64, 64, 64);">tags</font>`<font style="color:rgb(64, 64, 64);"> 文件。</font>
+ 找到tags文件

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743555487793-7591997b-d3a4-4cea-b934-9dda63a21572.png)

+ 追加相关内容的指令 `vi -t 内容`。
+ 继续向后追内容：ctrl + ]  ,这个ctrl是键盘左边的。
+ 退出追代码工具：底行模式下输入:q，回车。

---

## 2、使用fgetc（EOF）和fputc（EOF）

```plain
       #include <stdio.h>
       int fgetc(FILE *stream);
       功能：从指定文件中读取一个字符数据，并以无符号整数的形式返回
       参数：文件指针
       返回值：成功返回读取的字符对应的无符号整数，失败返回EOF并置位错误码

             #include <stdio.h>
       int fputc(int c, FILE *stream);
       功能：将指定的字符c写入到stream指向的文件中
       参数1：要写入的字符对应的无符号整数
       参数2：文件指针
       返回值：成功返回写入字符对的无符号整数，失败返回EOF并置位错误码
```

```cpp
#include<stdio.h>
int main(int argc, const char *argv[]){
    FILE *fp = NULL;
    if((fp = fopen("./file.txt", "w")) == NULL)
    {
        perror("fopen error");
        return -1;
    }

    fputc('H', fp);
    fputc('e', fp);
    fputc('l', fp);
    fputc('l', fp);
    fputc('o', fp);
    fclose(fp);
//此时光标在文字末尾
    if((fp = fopen("./file.txt", "r")) == NULL)
    {
        perror("fopen error");
        return -1;
    }

    int ch = 0;  // 改为 int 类型以正确检测 fgetc(fp)!=EOF
    //EOF是char类型
    while((ch = fgetc(fp)) != EOF)
    {
        printf("%c ", ch);
    }
    printf("\n");  // 添加换行

    fclose(fp);
    return 0;
}
```

+ 练习：cp srcfile destfile的功能实现
+ 外部传参 argc是参数个数，argv是内容

```cpp
#include<stdio.h>
int main(int argc, const char *argv[]){
    if(argc!=3){
        printf("input file error\n");
        printf("usage:./a.out srcfile destfile\n");
    }
    FILE* srcfp = NULL; //源文件指针
    FILE* destfp = NULL;//目标文件指针
    if((srcfp=fopen(argv[1],"r"))==NULL){
        perror("srcfile opren error");
        return -1;
    }
    if((destfp=fopen(argv[2],"w"))==NULL){
        perror("destfile open error");
        return -1;
    }
    char ch=0;
    while(1){
        ch=fgetc(srcfp);
        if(ch==EOF){
            break;
        }
        fputc(ch,destfp);
    }
    fclose(srcfp);
    fclose(destfp);
    return 0;
}
```

## 3、diff的使用

diff 文件 文件

比较它们的不同different，不同的内容打印出来

# 五、字符串读写：fgets（NULL）/fputs（EOF）

```cpp
       int fputs(const char *s, FILE *stream);
       功能：将指定的字符串，写入到指定的文件中
       参数1：要被写入的字符串
       参数2：文件指针
       返回值：成功返回本次写入字符的个数，失败返回EOF

       char *fgets(char *s, int size, FILE *stream);
       功能：从stream指向的文件中最多读取size-1个字符到s容器中，遇到回车或文件结束，会结束一
次读取，并且会将回车放入容器，最后自动加上一个字符串结束标识'\0'
       参数1：字符数组容器起始地址
       参数2：要读取的字符个数，最多读取size-1个字符
       参数3：文件指针
       返回值：成功返回容器s的起始地址，失败返回NULL

```

```cpp
#include<stdio.h>
#include<string.h> 
#include<iostream>
using namespace std;
int main(int argc, const char *argv[]){
    FILE* fp=NULL;
    if((fp=fopen("./file.txt","w"))==NULL){
        perror("fopen error");
        return -1;
    }
    //向文件中写入多个字符串
    char wbuf[123]="";
    size_t len =0;
    while(1){
        //从终端上输入字符
        cin>>wbuf;
        fputs(wbuf,fp);
        if(strcmp(wbuf,"quit")==0){
            break;
        }
    }
}
```

```cpp
#include<stdio.h>
#include<string.h> 
#include<iostream>
using namespace std;
int main(int argc, const char *argv[]){
    FILE* fp=NULL;
    if((fp=fopen("./file.txt","r"))==NULL){
        perror("fopen error");
        return -1;
    }
    //向文件中写入多个字符串
    char rbuf[123]="";
    size_t len =0;
    while(1){

        char* res=fgets(rbuf,sizeof(rbuf),fp);
        if(res==NULL){
            break;//读取结束
        }
    }
    printf("%s \n",rbuf);
}
```

---

+ cp 2个文件

```cpp
#include<iostream>
#include<stdio.h>
#include<string.h>
int main(int argc, char const *argv[])
{
    //判断外部 是否传入了2个文件
    if(argc!=3){
        printf("inut file error\n");
        printf("usage:./a.out srcfile destfile\n");
        return -1;
    }
    FILE* srcfp=NULL;//只读打开
    FILE* destfp = NULL;//只写打开
    if((srcfp=fopen(argv[1],"r"))==NULL){
        perror("fopen error");
        return -1;
    }
    if((destfp=fopen(argv[2],"w"))==NULL){
        perror("fopen error");
        return -1;
    }
    //定义搬运工
    char buf[128]="";
    while(1){
        //将数据从源文件中读取下来
        char* ptr=fgets(buf,sizeof(buf),srcfp);
        if(ptr==NULL){
            break;//读取结束
        }
        fputs(buf,destfp);
    }
    fclose(srcfp);
    fclose(destfp);
    return 0;
}

```

# 六、关于标准IO缓冲区问题

## 1、行缓存了解

+ 定义：和终端文件相关的缓冲区。
+ 行缓存的大小是1024字节。【满了就刷新】
+ 对应的文件指针：stdin、stdout

### 1.1、stdout

```cpp
#include<iostream>
#include<stdio.h>
using namespace std;
int main(int argc, char const *argv[])
{
    printf("行缓存：%ld\n",stdout->_IO_buf_end-stdout->_IO_buf_base);
    return 0;
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743660516142-08cc17e8-b8b8-43fc-a407-c64dadd852f5.png)

+ 会发现直接运行就是0缓存
+ 如果我复制printf，就变成1024了

```cpp
#include<iostream>
#include<stdio.h>
using namespace std;
int main(int argc, char const *argv[])
{
    printf("行缓存：%ld\n",stdout->_IO_buf_end-stdout->_IO_buf_base);
    printf("行缓存：%ld\n",stdout->_IO_buf_end-stdout->_IO_buf_base);
    return 0;
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743660650700-c6f43423-c2c8-41a8-930c-e0322efddbe0.png)

+ <font style="color:#DF2A3F;">如果缓存区没有被使用时，求出的大小为0，只有被至少使用一次后，缓存区的大小就被分配了</font>。

### 1.2、stdin

```cpp
#include<iostream>
#include<stdio.h>
using namespace std;
int main(int argc, char const *argv[])
{
    printf("行缓存：%ld\n",stdout->_IO_buf_end-stdout->_IO_buf_base);
    printf("行缓存：%ld\n",stdout->_IO_buf_end-stdout->_IO_buf_base);

    printf("行缓存：%ld\n",stdin->_IO_buf_end-stdin->_IO_buf_base);

    return 0;
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743661135573-29fc99fb-640e-4450-bcbb-7934f01c472c.png)

+ 没有使用标准输入就为0
+ 如果从终端输入一个整数让如num变量中，类似于cin>>num

```cpp
#include<iostream>
#include<stdio.h>
using namespace std;
int main(int argc, char const *argv[])
{
    printf("行缓存：%ld\n",stdout->_IO_buf_end-stdout->_IO_buf_base);
    printf("行缓存：%ld\n",stdout->_IO_buf_end-stdout->_IO_buf_base);

    printf("行缓存：%ld\n",stdin->_IO_buf_end-stdin->_IO_buf_base);
    int num=0;
    cin>>num;
    printf("行缓存：%ld\n",stdin->_IO_buf_end-stdin->_IO_buf_base);

    return 0;
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743661283795-94ae556f-aea9-4a1f-a225-836b0fc87106.png)

## 2、全缓存

+ 和<font style="color:#DF2A3F;">外界文件</font>相关的缓存区
+ 大小为<font style="color:#DF2A3F;">4096字节</font>
+ 对应的文件指针：fp

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743662018324-634a7aea-b646-43c6-8b2d-9afa009519e3.png)

```cpp
#include<iostream>
#include<stdio.h>
using namespace std;
int main(int argc, char const *argv[])
{
    printf("行缓存：%ld\n",stdout->_IO_buf_end-stdout->_IO_buf_base);
    printf("行缓存：%ld\n",stdout->_IO_buf_end-stdout->_IO_buf_base);

    printf("行缓存：%ld\n",stdin->_IO_buf_end-stdin->_IO_buf_base);
    int num=0;
    cin>>num;
    printf("行缓存：%ld\n",stdin->_IO_buf_end-stdin->_IO_buf_base);

    printf("------全缓存----------\n");
    FILE* fp=NULL;
    if((fp=fopen("./aa.txt","r"))==NULL){
        perror("fopen error\n");
        return -1;
    }
    //未使用全缓存
    printf("全缓存：%ld\n",fp->_IO_buf_end-fp->_IO_buf_base);
    //使用全缓存一次
    fgetc(fp);//从文件中读取一个字符
    printf("全缓存：%ld\n",fp->_IO_buf_end-fp->_IO_buf_base);

    return 0;
}
```



## 3、不缓存

+ 和标准出错相关的缓存区
+ 大小0字节
+ 对应的文件指针：stderr

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743662203520-07fe4c8a-cfae-4e0e-89da-c0a8320178f9.png)

```cpp
#include<iostream>
#include<stdio.h>
using namespace std;
int main(int argc, char const *argv[])
{
    printf("行缓存：%ld\n",stdout->_IO_buf_end-stdout->_IO_buf_base);
    printf("行缓存：%ld\n",stdout->_IO_buf_end-stdout->_IO_buf_base);

    printf("行缓存：%ld\n",stdin->_IO_buf_end-stdin->_IO_buf_base);
    int num=0;
    cin>>num;
    printf("行缓存：%ld\n",stdin->_IO_buf_end-stdin->_IO_buf_base);

    printf("------全缓存----------\n");
    FILE* fp=NULL;
    if((fp=fopen("./aa.txt","r"))==NULL){
        perror("fopen error\n");
        return -1;
    }
    //未使用全缓存
    printf("全缓存：%ld\n",fp->_IO_buf_end-fp->_IO_buf_base);
    //使用全缓存一次
    fgetc(fp);//从文件中读取一个字符
    printf("全缓存：%ld\n",fp->_IO_buf_end-fp->_IO_buf_base);
    fclose(fp);
    printf("------不缓存----------\n");

    printf("不缓存：%ld\n",stderr->_IO_buf_end-stderr->_IO_buf_base);
    perror("error");
    printf("不缓存：%ld\n",stderr->_IO_buf_end-stderr->_IO_buf_base);

    return 0;
}
```

## 4、行缓存区的刷新时机

### 4.1、缓冲区的时间没到，就不会打印数据

```cpp
#include<iostream>
#include<stdio.h>
int main(int argc, char const *argv[])
{
    //1、验证缓存区: 如果没有达到刷新时机，就不会讲数据进行刷新
    printf("hello world");//在终端上打印一个hello world
    //error会比hello world先打印出来
    perror("error\n");//在终端打印错误信息
    while(1){//阻塞程序不让进程结束


    }
    return 0;
}

/*
hello world没有到缓存区刷新时机，就不会输出数据
*/
```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743663689661-eb513e5f-3b84-40a1-b39d-c82b69cc3ce8.png)

### 4.2、程序结束会执行刷新缓存区

```cpp
#include<iostream>
#include<stdio.h>
int main(int argc, char const *argv[])
{

    //2、当程序结束后，会刷新缓存区
    printf("hello world");

    return 0;
}
```

### 4.3、当遇到换行会发生刷新缓存

```cpp
#include<iostream>
#include<stdio.h>
int main(int argc, char const *argv[])
{

    //2、当程序结束后，会刷新缓存区
    printf("hello world\n");

    return 0;
}

```

### 4.4、当输入输出发生切换时，也会刷新缓存

```cpp
#include<iostream>
#include<stdio.h>
int main(int argc, char const *argv[])
{

    // 4、当输入输出发生切换时，也会刷新缓存
    int num =0;
    printf("请输入>>>");//向标准输出缓存区中写入一组数据，没有换行符

    scanf("%d",&num);
    return 0;
}
```

### 4.5、当关闭缓存缓存对应的文件指针时，也会刷新缓存

+ 如果程序没结束，没换行，没输入输出切换

```cpp
#include<iostream>
#include<stdio.h>
int main(int argc, char const *argv[])
{
    //5、
    printf("hello world");//向标准输出缓存去写入数据，没有换行
    //fclose(stdout);//关闭标准输出指针
    while(1);
    return 0;
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743664836427-5eab862f-729b-428f-a97d-99963c2a549a.png)

+ 那么使用fclose关闭缓存相对应的文件指针时，也会刷新缓存。
+ <font style="color:#DF2A3F;">因为fclose里有个fflush函数，它刷新了缓存</font>。

```cpp
#include<iostream>
#include<stdio.h>
int main(int argc, char const *argv[])
{
    //5、
    printf("hello world");//向标准输出缓存去写入数据，没有换行
    fclose(stdout);//关闭标准输出指针
    while(1);
    return 0;
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743664996111-624889f1-0743-4581-9863-edb631da4392.png)

### 4.6、fflush**<font style="color:rgb(64, 64, 64);">强制刷新缓冲区</font>**<font style="color:rgb(64, 64, 64);"> 的函数（EOF）</font>

+ **<font style="color:rgb(64, 64, 64);">作用</font>**<font style="color:rgb(64, 64, 64);">：强制将缓冲区中</font>**<font style="color:rgb(64, 64, 64);">未写入的数据</font>**<font style="color:rgb(64, 64, 64);">立即提交到</font>**<font style="color:rgb(64, 64, 64);">目标设备</font>**<font style="color:rgb(64, 64, 64);">。</font>
+ <font style="color:#DF2A3F;">成功返回 0 ，失败返回 EOF </font><font style="color:rgb(64, 64, 64);">。</font>
+ man fflush会发现

```cpp
在介绍历是这样描述的
        For output streams, fflush() forces a write of all user-
       space buffered data  for  the  given  output  or  update
       stream via the stream's underlying write function.

       For  input streams associated with seekable files (e.g.,
       disk files, but not pipes or terminals),  fflush()  dis‐
       cards  any  buffered data that has been fetched from the
       underlying file, but has not been consumed by the appli‐
       cation.
对于输出流，fflush() 会强制将给定输出流或更新流中所有用户空间缓冲
            的数据，通过该流的底层写入函数立即写入目标设备。
对于与可寻址文件（如磁盘文件，但不包括管道或终端设备）关联的输入流，
            fflush() 会丢弃从底层文件中预读取但尚未被应用程序使用的任何缓冲数据。
```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743684777228-f3d76685-b521-4d81-978f-9916d1dd45b6.png)

+ 例子

```cpp
//6、fflush强制刷新缓存区，行缓存会被刷新
    printf("hello world");
    fflush(stdout);
    while(1){
    }
    return 0;
```

### 4.7、当缓冲区满了后，会刷新缓存，行缓存是1024字节

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743684932495-291695ad-fd67-438d-b426-9998c88e5ec2.png)

```cpp
//7、当缓冲区慢了后，会刷新缓存，行缓存是1024字节
    for(int i =0;i<1000;i++){
        printf("A");
    }
    while(1);
    return 0;
```

+ 改变的方法：<font style="color:#DF2A3F;">装满行缓存,但是会发现，下面代码的1025改成2000，也是打印1024个A。</font>

```cpp
//7、当缓冲区慢了后，会刷新缓存，行缓存是1024字节
    for(int i =0;i<1025;i++){
        printf("A");
    }
    while(1);//防止程序结束
    return 0;
```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743685050084-a59d4137-c79e-4c4f-8d1f-dd3eda55ce6a.png)

## 5、全缓存刷新时机

## 5.1、当缓冲区刷新时机未到时，不会刷新全缓存

+ 例子代码

```cpp
#include<iostream>
#include<stdio.h>
#include <errno.h>
using namespace std;
int main(int argc, char const *argv[])
{
    //打开文件
    FILE* fp =NULL;
    fp= fopen("./aa.txt","w+");
    if(fp==NULL){
        printf("open failsure failsure :%d\n",errno);
        return -1;
    }
    //1、当缓冲区刷新时机未到时，不会刷新全缓存
    fputs("hello world",fp);
    while (1);

    return 0;
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743689646741-6337f259-45f4-4bb5-9d1a-0a86d9be8a30.png)

## 5.2、换行会刷新

```cpp
//2、当换行时
    fputs("hello world\n",fp);
```

## 5.3、程序结束会刷新

```cpp
int main(){
    fputs("helloworld");
    return 0;
}
```

## 5.4、输入输出发生切换时会刷新

```cpp
fputs("hello world",fp);
fgetc(fp);
while(1);
```

### 5.5、当关闭缓冲区对应的文件指针时，也会刷新全缓存

```cpp
fputs("helloworld",fp);
fclose(fp);
while(1);
```

### 5.6、fflush（EOF）

```cpp
fputs("helloworld",fp);
fflush(fp);
while(1);
```

### 5.7、当缓存区满了后，再向缓冲区中存放数据会刷新全缓存

```cpp
#include<iostream>
#include<stdio.h>
#include <errno.h>
using namespace std;
int main(int argc, char const *argv[])
{
    //打开文件
    FILE* fp =NULL;
    fp= fopen("./aa.txt","w+");
    if(fp==NULL){
        printf("open failsure failsure :%d\n",errno);
        return -1;
    }
    for(int i=0;i<4096;i++){
        fputc('c',fp);
    }
    while(1);//4096就可以打印出来，但是4095就不行了
    return 0;
}

```

## 6、不缓存stderr

```cpp
#include<iostream>
#include<stdio.h>
#include <errno.h>
using namespace std;
int main(int argc, char const *argv[])
{
    //stderr会被刷新，缓存是0字节。
    //stdout就不会刷新了，缓存是4096。
    fputs("A",stderr);
    while(1);
    return 0;
}

```

# 七、格式化读写：fprintf/fscanf【外部和终端】

## 1、fprintf（-1EOF）

### 1.1、了解

+ 功能：**向指定的文件中输出一个格式串**
+ **参数1：****<font style="color:#DF2A3F;">文件指针</font>**
+ **参数2：****<font style="color:#DF2A3F;">格式串，</font>****可以包含格式控制符，例如：%d、%lf等等 。**
+ **参数3：****<font style="color:#DF2A3F;">可变参数</font>****<font style="color:rgb(51,51,51);">，</font>****<font style="color:#DF2A3F;">输出项列表</font>****<font style="color:rgb(51,51,51);">，参数个数由参数2中的格式控制符的个数决定</font>** 。
+ **返回值：成功返回是输出的字符个数，失败是 -1** EOF。

### 1.2、终端

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743697473317-00dfeea1-3f02-41aa-8fb4-cd07960346bf.png)

```cpp
#include<iostream>
#include<stdio.h>
using namespace std;
int main(int argc, char const *argv[])
{
    //向标准文件指针中写入数据----终端
    fprintf(stdout,"%d %lf %s\n",520,3.14,"星球");
    return 0;
}

```

### 1.3、外部文件

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743698426140-1997a6b2-afe7-4ec1-8d62-102d89b97f15.png)

```cpp
#include<iostream>
#include<stdio.h>
using namespace std;
int main(int argc, char const *argv[])
{
    //对外部文件进行格式化读写
    FILE* fp =NULL;
    fp=fopen("./a.txt","w");
    if(fp==NULL)return -1;
    //向文件中写入数据
    fprintf(fp,"%s %d","zyt",123456);
    char usrName[1024]="";//存放用户名
    int pwd ;//存放密码
    fscanf(fp,"%s %d",usrName,&pwd);
    //关闭文件
    fclose(fp);
    printf("pwd=%d userName=%s\n",pwd,usrName);
    return 0;
}

```

## 2、fscanf（EOF）

### 2.1、了解

+ 功能：**向指定的文件中输入一个格式串**
+ **参数1：****<font style="color:#DF2A3F;">文件指针</font>**
+ **参数2：****<font style="color:#DF2A3F;">格式串</font>****，可以包含格式控制符，例如：%d、%lf等等 。**
+ **参数3：****<font style="color:#DF2A3F;">可变参数</font>****<font style="color:rgb(51,51,51);">，</font>****<font style="color:#DF2A3F;">输入项地址列表</font>****<font style="color:rgb(51,51,51);">，参数个数由参数2中的格式控制符的个数决定</font>**。
+ **返回值：****<font style="color:rgb(51,51,51);">成功返回读入的项数，失败返回EOF并置位错误码</font>**。

### 2.2、终端

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743697630234-f7dc3174-5222-4146-9627-d7c0e92cff94.png)

```cpp
#include<iostream>
#include<stdio.h>
using namespace std;
int main(int argc, char const *argv[])
{
    //向标准文件指针中写入数据
    fprintf(stdout,"%d %lf %s\n",520,3.14,"星球");

    //向标准输入中读取数据到程序中来
    int num =0;
    fscanf(stdin,"%d",&num);
    printf("num=%d\n",num);
    return 0;
}

```

### 2.3、外部文件

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743698467362-a2b4e077-3a47-49ff-8c7a-327444c5d0a8.png)

```cpp
#include<iostream>
#include<stdio.h>
using namespace std;
int main(int argc, char const *argv[])
{
    //对外部文件进行格式化读写
    FILE* fp =NULL;
    fp=fopen("./a.txt","w");
    if(fp==NULL)return -1;
    //向文件中写入数据
    fprintf(fp,"%s %d","zyt",123456);
    char usrName[1024]="";//存放用户名
    int pwd ;//存放密码
    fscanf(fp,"%s %d",usrName,&pwd);
    //关闭文件
    fclose(fp);
    printf("pwd=%d userName=%s\n",pwd,usrName);
    return 0;
}

```

## 3、练习

+ <font style="color:rgb(51,51,51);">使用fprintf和fscanf完成注册和登录功能，要求做个小菜单，三个功能，功能1是注册，用户输入账号和 密码后，将其存放到外部文件中；功能2是登录，用户输入登录账号和密码后，如果跟文件中匹配，则 提示登陆成功，如果全部不匹配，则提示登录失败；功能0，表示退出。菜单可以循环调用</font>

```cpp
#include<iostream>
#include<stdio.h>
#include<string.h>
using namespace std;
//定义注册函数
int do_register(){
    char reg_name[20]="";
    char reg_pwd[20]="";
    printf("请输入账户:");
    fgets(reg_name,sizeof(reg_name),stdin);
    reg_name[strlen(reg_name)-1]='\0';//把换行符换成结束符
    printf("请输入注册密码");
    fgets(reg_pwd,sizeof(reg_pwd),stdin);
    reg_pwd[strlen(reg_pwd)-1]='\0';
    //将账户和密码写入文件
    FILE* wfp = NULL;
    wfp=fopen("./user.txt","a");//追加的形式，避免w的清空
    if(wfp==NULL){
        perror("fopen error");
        return -1;
    }
    //没问题，写入
    fprintf(wfp,"%s %s\n",reg_name,reg_pwd);
    fclose(wfp);
    printf("注册成功\n");
    return 1;
}
int do_login()
{
    //定义容器存储登录账户和密码
    char log_name[20]="";
    char log_pwd[20]="";
    char name[20]="";
    char pwd[20]="";
    //提示并输入登录账户和密码
    printf("请输入登录账户:");
    fgets(log_name,sizeof(log_name),stdin);
    log_name[strlen(log_name)-1]='\0';
    printf("请输入登录密码:");
    fgets(log_pwd,sizeof(log_pwd),stdin);
    log_pwd[strlen(log_pwd)-1]='\0';
    //打开文件
    FILE* rfp = NULL;
    if((rfp=fopen("./user.txt","r"))==NULL){
        perror("fopen error");
        return -1;
    }
    //将登录账户和密码跟文件中所有账户和密码进行匹配
    while(1)
    {
        //将每一行的账户和密码读取出来
        int res=fscanf(rfp,"%s %s",name,pwd);
        if(res==EOF){
            break;
        }
        //将读取出来的账户和密码跟登录账户和密码进行匹配
        if(strcmp(name,log_name)==0&&strcmp(pwd,log_pwd)==0){
            printf("登录成功\n");
            fclose(rfp);
            return 1;
        }
    }
    //关闭文件
    fclose(rfp);
    printf("登录失败,请重新登录\n");
    return 0;
}
int main(int argc,const char* argv[])
{
    int menu =0;//存储菜单选项变量
    while(1){
        system("clear");//创建子进程调用终端指令
        printf("\t\t====XXX登录界面====\n");
        printf("\t\t====1、注册====\n");
        printf("\t\t====2、登录====\n");
        printf("\t\t====0、退出====\n");
        printf("输入的功能选项:");
        scanf("%d",&menu);
        getchar();//getchar吸收一下回车
        switch(menu){
        case 1:
            //注册
            do_register();
            break;
        case 2:
            //登录
            int res;
            res=do_login();
            if(res==1){
                printf("登录成功后的界面\n");
            }
            break;
        case 0:
            exit(EXIT_SUCCESS);
        default :
            printf("重新登录\n");
        }
        printf("请按回车清屏");
        while(getchar() != '\n');
    }
    return -1;
}
```

# 八、格式串转字符串存入字符数组中：sprintf（EOF）/snprintf（EOF）

## 1、了解

```cpp
       int sprintf(char *str, const char *format, ...);
       功能：将指定的格式串转换为字符串，放入字符数组中
       参数1：字符数组的起始地址
       参数2：格式串，可以包含多个格式控制符
       参数3：可变参数
       返回值：成功返回转换的字符个数，失败返回EOF

       对于上述函数而言，用一个小的容器去存储一个大的转换后的字符时，会出现指针越界的段错误，为
了安全起见，引入了snprintf

       int snprintf(char *str, size_t size, const char *format, ...);
       功能：将格式串中最多size-1个字符转换为字符串，存放到字符数组中
       参数1：字符数组的起始地址
       参数2：要转换的字符个数，最多为size-1
       参数3：格式串，可以包含多个格式控制符
       参数4：可变参数
       返回值：如果转换的字符小于size时，返回值就是成功转换字符的个数，如果大于size则只转换
size-1个字符，返回值就是size，失败返回EOF

```

## 2、sprintf

+ 如果字符容器小了，会越界 ，例如：

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743758946523-9d8bc64d-1b41-4a8b-ba0f-c4ccb31a5b36.png)

```cpp
#include<iostream>
#include<stdio.h>
using namespace std;
int main(int argc, char const *argv[])
{
    char buf[10]="";//定义一个字符串容器
    sprintf(buf,"%d %s %lf",100,"zhangsanfeng",10.5);
    //输出转换 后的字符串
    printf("buf = %s \n",buf);
    return 0;
}

```

## 3、snprintf

+ 例子

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743759395948-1c3fa259-fcd8-45c5-8732-786d324a5519.png)

```cpp
#include<iostream>
#include<stdio.h>
using namespace std;
int main(int argc, char const *argv[])
{
    char buf[10]="";//定义一个字符串容器
    snprintf(buf,sizeof(buf),"%d %s %lf",100,"zhangsanfeng",10.5);
    //输出转换 后的字符串
    printf("buf = %s \n",buf);
    return 0;
}

```

# 九、模块化读写 （fread/fwrite）

# 1、了解

<details class="lake-collapse"><summary id="u6654c8f0"><span class="ne-text">了解</span></summary><p id="u9119bf64" class="ne-p"><span class="ne-text" style="color: rgb(0,136,85)">size_t </span><span class="ne-text" style="color: rgb(0,0,255)">fread</span><span class="ne-text" style="color: rgb(51,51,51)">(</span><span class="ne-text" style="color: rgb(0,136,85)">void </span><span class="ne-text" style="color: rgb(0,0,0)">ptr</span><em><span class="ne-text" style="color: rgb(51,51,51)">, </span></em><span class="ne-text" style="color: rgb(0,136,85)">size_t </span><span class="ne-text" style="color: rgb(0,0,0)">size</span><span class="ne-text" style="color: rgb(51,51,51)">, </span><span class="ne-text" style="color: rgb(0,136,85)">size_t </span><span class="ne-text" style="color: rgb(0,0,0)">n</span><em><span class="ne-text" style="color: rgb(0,0,0)"> </span></em><span class="ne-text" style="color: rgb(0,0,0)">memb</span><em><span class="ne-text" style="color: rgb(51,51,51)">, </span></em><span class="ne-text" style="color: rgb(0,0,0)">FILE</span><span class="ne-text" style="color: rgb(152,26,26)"> </span><span class="ne-text" style="color: rgb(0,0,0)">stream</span><span class="ne-text" style="color: rgb(51,51,51)">);       </span></p><p id="u05a50c40" class="ne-p"><span class="ne-text" style="color: rgb(0,0,0)">功能：从stream指向的文件中，读取nmemb项数据，每一项的大小为size，将整个结果放入ptr指 向的容器中 </span><span class="ne-text" style="color: rgb(51,51,51)">       </span></p><p id="u0c13453f" class="ne-p"><span class="ne-text" style="color: rgb(0,0,0)">参数1：容器指针，是一个void</span><span class="ne-text" style="color: rgb(152,26,26)">*</span><span class="ne-text" style="color: rgb(0,0,0)">类型，表示可以存储任意类型的数据 </span><span class="ne-text" style="color: rgb(51,51,51)">       </span></p><p id="u961c973b" class="ne-p"><span class="ne-text" style="color: rgb(0,0,0)">参数2：要读取数据每一项的大小 </span><span class="ne-text" style="color: rgb(51,51,51)">       </span></p><p id="ua72d16a0" class="ne-p"><span class="ne-text" style="color: rgb(0,0,0)">参数3：要读取数据的项数 </span><span class="ne-text" style="color: rgb(51,51,51)">       </span></p><p id="u887145b0" class="ne-p"><span class="ne-text" style="color: rgb(0,0,0)">参数4：文件指针 </span><span class="ne-text" style="color: rgb(51,51,51)">       </span></p><p id="uf445e49a" class="ne-p"><span class="ne-text" style="color: rgb(0,0,0)">返回值：成功返回nmemb</span><span class="ne-text" style="color: rgb(51,51,51)">,</span><span class="ne-text" style="color: rgb(0,0,0)">就是成功读取的项数，失败返回小于项数的值，或者是0</span></p><p id="u57eb9f8b" class="ne-p"><span class="ne-text" style="color: rgb(0,0,0)"></span></p><p id="u66878d6f" class="ne-p"><span class="ne-text" style="color: rgb(0,136,85)">size_t </span><span class="ne-text" style="color: rgb(0,0,255)">fwrite</span><span class="ne-text" style="color: rgb(51,51,51)">(</span><span class="ne-text" style="color: rgb(119,0,136)">const </span><span class="ne-text" style="color: rgb(0,136,85)">void* </span><span class="ne-text" style="color: rgb(0,0,0)">ptr</span><span class="ne-text" style="color: rgb(51,51,51)">, </span><span class="ne-text" style="color: rgb(0,136,85)">size_t </span><span class="ne-text" style="color: rgb(0,0,0)">size</span><span class="ne-text" style="color: rgb(51,51,51)">, </span><span class="ne-text" style="color: rgb(0,136,85)">size_t </span><span class="ne-text" style="color: rgb(0,0,0)">nmemb</span><span class="ne-text" style="color: rgb(51,51,51)">, </span><span class="ne-text" style="color: rgb(0,0,0)">FILE</span><span class="ne-text" style="color: rgb(152,26,26)">* </span><span class="ne-text" style="color: rgb(0,0,0)">stream</span><span class="ne-text" style="color: rgb(51,51,51)">);       </span></p><p id="u63644a52" class="ne-p"><span class="ne-text" style="color: rgb(0,0,0)">功能：向stream指向的文件中写入nmemb项数据，每一项的大小为size，数据的起始地址为ptr</span></p><p id="u797a9a7e" class="ne-p"><span class="ne-text" style="color: rgb(0,0,0)">。</span><strong><span class="ne-text" style="color: #DF2A3F">但是它最终读取到的不一定是字符串。就可以配合使用fwrite</span></strong><span class="ne-text" style="color: rgb(0,0,0)">。</span></p><p id="u5c8467a2" class="ne-p"><span class="ne-text" style="color: rgb(0,0,0)">参数1：要写入数据的起始地址 </span><span class="ne-text" style="color: rgb(51,51,51)">       </span></p><p id="u303db50a" class="ne-p"><span class="ne-text" style="color: rgb(0,0,0)">参数2：每一项的大小 </span><span class="ne-text" style="color: rgb(51,51,51)">       </span></p><p id="ub425df9e" class="ne-p"><span class="ne-text" style="color: rgb(0,0,0)">参数3：要写入的总项数 </span><span class="ne-text" style="color: rgb(51,51,51)">       </span></p><p id="u60d39d6e" class="ne-p"><span class="ne-text" style="color: rgb(0,0,0)">参数4：文件指针 </span><span class="ne-text" style="color: rgb(51,51,51)">       </span></p><p id="ueb8371f3" class="ne-p"><span class="ne-text" style="color: rgb(0,0,0)">返回值：成功返回写入的项数，失败返回小于项数的值，或者是0</span></p></details>
# 2、fwrite和换行处理
```cpp
#include<iostream>
#include<stdlib.h>
#include<string.h>
using namespace std;
int main(int argc, char const *argv[])
{
    //定义文件指针，以只写的形式打开文件
    FILE* fp=NULL;
    if((fp=fopen("./test.txt","w"))==NULL){
        perror("fopen error");
        return -1;
    }
    char wbuf[128]="";
    while(1){
        //提示从终端上输入字符串
        printf("请输入>>>");
        fgets(wbuf,sizeof(wbuf),stdin);//从标准输入中读取数据
        wbuf[strlen(wbuf)-1]='\0';

        //判断输入是否为quit
        if(strcmp(wbuf,"quit")==0){
            break;
        }
        //将字符串写入文件
        fwrite(wbuf,strlen(wbuf),1,fp);
        //以每个字符串为单位写入文件fp，每次写一项
        //刷新缓存区
        fflush(fp);
        printf("录入成功\n");
    }
    
    fclose(fp);
    return 0;

}

```

+ **<font style="color:#DF2A3F;">因为wbuf的最后是 \n字符，如果处理成 \0(wbuf[strlen-1]='\0')就没有换行了。就需要手动添加换行符 </font>**
+ **<font style="color:#DF2A3F;">如果我不换成\0，就存在\n了。不过在strcmp匹配的时候，要加上\n，因为字符的后面有这个符号</font>**

```cpp
#include<iostream>
#include<stdlib.h>
#include<string.h>
using namespace std;
int main(int argc, char const *argv[])
{
    //定义文件指针，以只写的形式打开文件
    FILE* fp=NULL;
    if((fp=fopen("./test.txt","w"))==NULL){
        perror("fopen error");
        return -1;
    }
    char wbuf[128]="";
    while(1){
        //提示从终端上输入字符串
        printf("请输入>>>");
        fgets(wbuf,sizeof(wbuf),stdin);//从标准输入中读取数据
        //wbuf[strlen(wbuf)-1]='\0';

        //判断输入是否为quit
        if(strcmp(wbuf,"quit\n")==0){
            break;
        }
        //将字符串写入文件
        fwrite(wbuf,strlen(wbuf),1,fp);
        //以每个字符串为单位写入文件fp，每次写一项
        //fwrite(wbuf,strlen(wbuf),1,fp);//放入/n
        //每输入一个字符串加上一个换行
        //刷新缓存区
        fflush(fp);
        printf("录入成功\n");
    }

    fclose(fp);
    return 0;
}
```

# 3、fread处理（重点，颠覆认知）

+ **<font style="color:#DF2A3F;">fread的读取与换行\n 无关【会读取换行】。而且可能是二进制类型的内容。用fwrite读取</font>**。

```cpp
#include<iostream>
#include<stdlib.h>
#include<string.h>
using namespace std;
int main(int argc, char const *argv[])
{
    //定义文件指针，以只写的形式打开文件
    FILE* fp=NULL;
    if((fp=fopen("./test.txt","w"))==NULL){
        perror("fopen error");
        return -1;
    }
    char wbuf[128]="";
    while(1){
        //提示从终端上输入字符串
        printf("请输入>>>");
        fgets(wbuf,sizeof(wbuf),stdin);//从标准输入中读取数据
        //wbuf[strlen(wbuf)-1]='\0';

        //判断输入是否为quit
        if(strcmp(wbuf,"quit\n")==0){
            break;
        }
        //将字符串写入文件
        fwrite(wbuf,strlen(wbuf),1,fp);
        //以每个字符串为单位写入文件fp，每次写一项
        //fwrite(wbuf,strlen(wbuf),1,fp);//放入/n
        //每输入一个字符串加上一个换行
        //刷新缓存区
        fflush(fp);
        printf("录入成功\n");
    }

    fclose(fp);
    if((fp==fopen("./test.txt","r"))==NULL){
        perror("fopen error");
        return -1;
    }
    char rbuf[10]="";
    int res=fread(rbuf,1,sizeof(rbuf),fp);
    fwrite(rbuf,1,res,stdout);//向标准输出中写入数据
    fclose(fp);

    return 0;
}

```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743783343113-0dc9540d-076e-49c1-b157-7d8d0e98d01b.png)

# 3、整数的读写【重点】

+ **<font style="color:#DF2A3F;">可以用于整数读取，二进制文件内容</font>**

```cpp
#include<iostream>
#include<stdlib.h>
#include<string.h>
using namespace std;
int main(int argc, char const *argv[])
{
    //定义文件指针，以只写的形式打开文件
    FILE* fp=NULL;
    if((fp=fopen("./test.txt","w"))==NULL){
        perror("fopen error");
        return -1;
    }


    //定义要写入文件中的数据
    int num =16;
    //将数据写入文件中
    fwrite(&num,sizeof(num),1,fp);//将num中的数据写入fp指向的文件中
    //关闭文件
    fclose(fp);
    if((fp=fopen("./test.txt","r"))==NULL){
        perror("fopen error");
        return -1;
    }
    //定义接收的变量
    int key =0;
    //从文件中读取数据
    fread(&key,sizeof(key),1,fp);


    fclose(fp);
    printf("key= %d \n",key);
    return 0;
}
```

# 4、结构体的读写【重点】

+ 可以用于结构体struct或class都行

```cpp
#include<iostream>
#include<stdlib.h>
#include<string.h>
using namespace std;
class Stu{
public:
    char name[20];//性别
    int age;//年龄
    double score;//成绩
};
int main(int argc, char const *argv[])
{
    //定义文件指针，以只写的形式打开文件
    FILE* fp=NULL;
    if((fp=fopen("./test.txt","w"))==NULL){
        perror("fopen error");
        return -1;
    }

    Stu s[3]={
        {"张三",18,98},
        {"李四",20,88},
        {"王五",16,95}
    };
    fwrite(s,sizeof(Stu),3,fp);

    //关闭文件
    fclose(fp);
    if((fp=fopen("./test.txt","r"))==NULL){
        perror("fopen error");
        return -1;
    }
    //定义一个对象接收读取的结果
    Stu temp;
    fread(&temp,sizeof(Stu),1,fp);
    printf("name:%s ,age:%d,score:%.2lf\n",temp.name,temp.age,temp.score);
    fclose(fp);

    return 0;
}
```

# 十、关于文件光标：fseek(-1错误码)/ftell（-1错误码）/rewind（无）

```cpp
       int fseek(FILE *stream, long offset, int whence);
       功能：移动文件光标位置，将光标从指定位置处进行前后偏移
       参数1：文件指针
       参数2：偏移量
           >0:表示从指定位置向后偏移n个字节
           <0:表示从指定位置向前偏移n个字节
           =0:在指定位置处不偏移
       参数3：偏移的起始位置
           SEEK_SET：文件起始位置
           SEEK_CUR：文件指针当前位置
           SEEK_END：文件结束位置
       返回值：成功返回0，失败返回-1并置位错误码
       long ftell(FILE *stream);
       功能：获取文件指针当前的偏移量
       参数：文件指针
       返回值：成功返回文件指针所在的位置，失败返回-1并置位错误码
       eg: fseek(fp, 0, SEEK_END);        //将光标定位在结尾
         ftell(fp);                //该函数的返回值，就是文件大小
       void rewind(FILE *stream);
       功能：将文件光标定位在开头：fseek(fp, 0, SEEK_SET);
       参数：文件指针
       返回值：无
```

+ 读取文件的大小

```cpp
#include<iostream>
#include<stdlib.h>
#include<string.h>
using namespace std;
class Stu{
public:
    char name[20];//性别
    int age;//年龄
    double score;//成绩
};
int main(int argc, char const *argv[])
{
    //定义文件指针，以只写的形式打开文件
    FILE* fp=NULL;
    if((fp=fopen("./test.txt","w"))==NULL){
        perror("fopen error");
        return -1;
    }

    Stu s[3]={
        {"张三",18,98},
        {"李四",20,88},
        {"王五",16,95}
    };
    fwrite(s,sizeof(Stu),3,fp);
    printf("此时文件的大小为%ld\n",ftell(fp));//文件大小
    //关闭文件
    fclose(fp);
    if((fp=fopen("./test.txt","r"))==NULL){
        perror("fopen error");
        return -1;
    }
    //定义一个对象接收读取的结果
    Stu temp;
    //从文件中读取一个学生的信息
    fread(&temp,sizeof(Stu),1,fp);
    printf("name:%s ,age:%d,score:%.2lf\n",temp.name,temp.age,temp.score);
    fclose(fp);

    return 0;
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743816866964-836c2a51-6886-4240-a9f4-7d33e6cfc6c1.png)

+ 偏移

```cpp
#include<iostream>
#include<stdlib.h>
#include<string.h>
using namespace std;
class Stu{
public:
    char name[20];//性别
    int age;//年龄
    double score;//成绩
};
int main(int argc, char const *argv[])
{
    //定义文件指针，以只写的形式打开文件
    FILE* fp=NULL;
    if((fp=fopen("./test.txt","w"))==NULL){
        perror("fopen error");
        return -1;
    }

    Stu s[3]={
        {"张三",18,98},
        {"李四",20,88},
        {"王五",16,95}
    };
    fwrite(s,sizeof(Stu),3,fp);

    //将光标移动到开头位置
    //fseek(fp,0,SEEK_SET);
    //将光标直接定位到第2个学生信息前，但是此时光标在最后
    //fseek(fp,sizeof(Stu),SEEK_SET);//将光标从开头后移一个学生空间的内容
    fseek(fp,-sizeof(Stu)*2,SEEK_CUR);//将光标从当前位置向前偏移2个学生空间的内容

    //定义一个对象接收读取的结果
    Stu temp;
    //从文件中读取一个学生的信息
    fread(&temp,sizeof(Stu),1,fp);
    printf("name:%s ,age:%d,score:%.2lf\n",temp.name,temp.age,temp.score);
    fclose(fp);

    return 0;
}
```
