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

- 例子

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
#define ETXTBSY         26     /* Text file busy */
#define EFBIG           27     /* File too large */
#define ENOSPC         28     /* No space left on device */
#define ESPIPE         29     /* Illegal seek */
```

- 0表示**成功**

- 2表示**文件或目录不存在**

- 关于错误码的处理 函数(strerror 、perror)

```cpp
使用man perror找到它的文章

#include <errno.h>   //错误码所在的头文件，里面定义了一个全局变量
int errno;


```

<img src="file:///C:/Users/LEGION/Pictures/d425f9c5-91d5-4964-a5e7-ac829668887f.png" title="" alt="d425f9c5-91d5-4964-a5e7-ac829668887f" style="zoom:67%;">

- 打印错误码

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

- 打印错误码并显示错误码的信息

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

- 例如

```cpp
#include"stdio.h"        //标准的输入输出头文件
#include<errno.h>          //错误码所在的头文件
#include<string.h>         //字符串处理的头文件
int main(int argc, const char *argv[])
{
    //1、定义一个文件指针
    FILE *fp = NULL;
    fp = fopen("./file.txt", "r");           
    if(fp == NULL)
   {
        //printf("fopen error: %d, errmsg:%s\n", errno, strerror(errno));     //
打印错误信息，并输出错误码 2
        perror("fopen error");      //打印当前错误码对应的错误信息
        return -1;
   }
    printf("fopen success\n");
    //2、关闭文件
    fclose(fp);
    
    return 0;
}
```

# 四、单字符读写：fputc/fgetc











# 五、字符串读写：fgets/fputs











# 六、格式化读写：fprintf/fscanf











# 七、格式串转字符串存入字符数组中：sprintf/snprintf





# 八、模块化读写 （fread/fwrite）









# 九、关于文件光标：fseek/ftell/rewind














































































