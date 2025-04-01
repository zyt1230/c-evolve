# 一、标准IO与文件IO的区别

## 1.1 IO概念

1> IO：顾名思义就是输入输出，程序与外部设备进行信息交换的过程称为IO操作 

2> 最先接触的IO：#include<stdio> 标准的缓冲输入输出头文件 

## 1.2 IO的分类

1> **标准IO**：使用系统提供的**库函数**实现 

2> **文件IO**：基于**系统调用完成**，每进行一次系统调用，进程会从用户空间向内核空间进行一次切换，当用户空间与内核空间进行切换时，进程就会进入一次挂起状态，从而导致进程执行效率低 。

3> **文件IO与标准IO的区别**：**标准IO**相比于文件IO而言，**提供了缓冲区**，用户可以将数据先放入缓冲区中，等到缓冲区时机到了后，统一进行一次系统调用，将数据刷入内核空间。

![63aaf79a-daeb-46f1-a23a-4ad85456dbfd](file:///C:/Users/LEGION/Pictures/63aaf79a-daeb-46f1-a23a-4ad85456dbfd.png)

4> 标准IO接口：printf/scanf、fopen/fclose、fgetc/fputc、fgets/futs、fprintf/fscaf、 fread/fwrite、fseek/ftell/rewind 

 文件IO接口：open/close、read/write、lseek

![168ae85a-726a-45c4-9d2c-f177c7b089b9](file:///C:/Users/LEGION/Pictures/168ae85a-726a-45c4-9d2c-f177c7b089b9.png)

# 二、标准IO的File结构体

> File用于描述一个文件全部信息的结构体

```cpp
struct FILE
{
 char * _IO_buf_base;       //缓冲区的起始地址
 char * _IO_buf_end;         //缓冲区终止地址

 int _fileno;               //文件描述符，用于进行系统调用
};


```

特殊的FILE指针，这三个指针，全部都是针对于终端文件而言的，当程序启动后，系统默认打开的

> 三个特殊文件指针
>  stderr: 标准出错指针
>  stdin: 标准输入指针
>  stdout: 标准输出指针
