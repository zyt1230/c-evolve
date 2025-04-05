# 一、了解
+ **<font style="color:#DF2A3F;">文件IO就是用系统调用（系统调用就是内核提供的函数）实现，只要使用了文件IO接口，进程就会从用户空间向内核空间进行一次切换。该操作没有缓存区</font>**。
+ **<font style="color:#DF2A3F;">标准IO的实现，也是内部调用了文件IO操作</font>**。
+ man 2 open  查看头文件

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743833950417-b956408c-aaeb-4204-9795-6c490b964ac3.png)

# 二、文件描述符
+ 本质：文件描述符是一个<font style="color:#DF2A3F;">大于等于0的整数</font>，在使用open函数打开文件时，就会产生一个<font style="color:#DF2A3F;">用于操作文件的句柄</font>，这就是文件描述符。
+ 在一个进程中，能够打开的文件描述符是有限制的，一般是1024个，<font style="color:#DF2A3F;">[0 , 1023]可以通过指令ulimit -a查看</font>。可以使用`ulimit -n 数字`进行修改大小，改成`数字`。
+ ulimit -a ：<font style="color:rgb(64, 64, 64);">执行后会显示当前 shell 进程及其子进程能</font><font style="color:#DF2A3F;">使用的系统资源上限</font><font style="color:rgb(64, 64, 64);"> 。</font>

```cpp
各参数含义：
选项	资源限制	说明
-c	core file size	核心转储文件大小（0 表示禁止生成 core 文件）
-d	data seg size	进程数据段（Data Segment）大小
-e	scheduling priority	进程调度优先级（-20 最高，19 最低）
-f	file size	进程能创建的文件大小
-i	pending signals	待处理信号数量上限
-l	max locked memory	进程可锁定的内存大小
-m	max memory size	进程物理内存使用上限
-n	open files	进程能打开的文件描述符数量（常见调优项）
-p	pipe size	管道缓冲区大小（单位：512 字节）
-q	POSIX message queues	POSIX 消息队列大小
-r	real-time priority	实时调度优先级
-s	stack size	进程栈大小（单位：KB）
-t	cpu time	进程占用 CPU 时间上限
-u	max user processes	用户能创建的进程数上限
-v	virtual memory	进程虚拟内存大小
-x	file locks	文件锁数量
```

+ 文件描述符的使用原则一般是**<font style="color:#DF2A3F;">最小 未 分配原则</font>**。比如：1 2 3 4 6都被用了，下一个会自动调用5 。
+ <font style="color:#DF2A3F;">特殊的文件描述符：0、1、2 </font>，这个三个文件描述符在一个进程启动时就默认打开了，分别是标准输入，标准输出，标准出错 。

```cpp
#include<iostream>
#include<stdlib.h>
#include<string.h>
using namespace std;
int main(int argc, char const *argv[])
{
    //分别输出 标准输入，标准输出，标准出错的文件描述符
    printf("stdin->_fileno=%d\n",stdin->_fileno);   //0
    printf("stdout->_fileno=%d\n",stdout->_fileno); //1
    printf("stderr->_fileno=%d\n",stderr->_fileno); //2
}

```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743819289074-d9b2c991-bc46-4ae0-8072-66a6c253badc.png)

# 三、打开文件：open（-1错位码）
```cpp
       #include <sys/types.h>
       #include <sys/stat.h>
       #include <fcntl.h>
       int open(const char *pathname, int flags);
       int open(const char *pathname, int flags, mode_t mode);
       功能：打开或创建一个文件，并返回该文件的文件描述符，返回文件描述符的规则是最小未分配原则
       参数1：要打开的文件文件路径
       参数2：打开模式
       以下三种方式必须选其一：O_RDONLY(只读)、O_WRONLY（只写）、O_RDWR（读写）
       以下的打开方式可以有零个或多个，跟上述的方式一起用位或运算连接到一起
       O_CREAT:用于创建文件，如果文件不存在，则创建文件，如果文件存在，则打开文件，如果
flag中包含了该模式，则函数的第三个参数必须要加上
       O_APPEND:以追加的形式打开文件，光标定位在结尾
       O_TRUNC:清空文件
       O_EXCL:常跟O_CREAT一起使用，确保本次要创建一个新文件，如果文件已经存在，则open
函数报错
       eg:
       "w": O_WRONLY|O_CREAT|O_TRUNC
       "w+": O_RDWR|O_CREAT|O_TRUNC
       "r": O_RDONLY
       "r+": O_RDWR
       "a": O_WRONLY|O_CREAT|O_APPEND
       "a+": O_RDWR|O_CREAT|O_APPEND
       参数3：如果参数2中的flag中有O_CREAT时，表示创建新文件，参数3就必须给定，表示新创建的
文件的权限
       
       当前终端的umask的值，可以通过指令 umask 来查看，一般默认为为 0022，表示当
前进程所在的组中对该文件没有 写权限，其他用户对该文件也没有 写权限
       当前终端的umask的值是可以更改的，通过指令：umask 数字进行更改，这种方式只对
当前终端有效
       权限	说明
        0644	用户可读写，其他用户只读
        0755	用户可读写执行，其他用户可读执行
        0600	仅用户可读写
        0777    最大权限
       注意：如果不给权限，那么当前创建的权限会是一个随机值
       注意: 最终权限受 umask 影响（实际权限 = mode & ~umask）。所以可以改umask的值。
    权限	字符表示 	数字值
    读	      r	    	 4
    写	      w	   		 2
    执行	  x			 1
    无权限	  -			 0
       返回值：成功返回打开文件的文件描述符，失败返回-1并置位错误码
```

+ man fopen 里面有这个表格

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743820426718-42b54e51-41c0-4c58-a121-a4a7d52a8094.png)

```cpp
#include<myhead.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
int main(int argc,const char* argv[])
{
    //1、定义文件描述符，对于文件IO而言，句柄就是文件描述符
    int fd=-1;
    if((fd=open(".tt.txt",O_CREAT|O_WRONLY,0777))==-1){
        perror("open error");
        return -1;
    }
    printf("open success fd = %d\n",fd);//fd=3，因为012已经被分配

    return 0;
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743836833094-ab66a082-6590-4568-b8a8-699ac27c425b.png)

+ 会发现tt.txt 是r的权限
+ 如果删掉tt.txt，重新编译执行，权限变 wS了。说明是随机的。需要我们设置权限。

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743836910843-77bcced3-ab12-481f-afcf-8120f4fa707a.png)

+ **<font style="color:#DF2A3F;">改变umask</font>**

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743837678353-de8db92c-ff1b-4eb7-a472-33308f52d2db.png)

# 四、文件关闭：close（-1错位码）
```cpp
       #include <unistd.h>
       int close(int fd);
       功能：关闭文件描述符对应的文件
       参数：文件描述符
       返回值：成功返回0，失败返回-1并置位错误码
```

```cpp
#include<myhead.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
#include<unistd.h>
int main(int argc,const char* argv[])
{
    //1、定义文件描述符，对于文件IO而言，句柄就是文件描述符
    int fd=-1;
    if((fd=open("./tt.txt",O_CREAT|O_WRONLY|0777))==-1){
        perror("open error");
        return -1;
    }
    printf("open success fd = %d\n",fd);//fd=3，因为012已经被分配
    close(fd);

    return 0;
}
```

# 五、读写操作：read（-1错位码）、write（-1错位码）
```cpp
       #include <unistd.h>
       ssize_t read(int fd, void *buf, size_t count);
       功能：从fd文件描述符引用的文件中读取count的字符放入buf对应的容器中
       参数1：已经打开文件对应的文件描述符
       参数2：容器的起始地址
       参数3：要读取的字符个数
       返回值：成功返回读取的字符个数，这个个数可能会小于count的值，失败返回-1并置位错误码
       
       #include <unistd.h>
       ssize_t write(int fd, const void *buf, size_t count);
       功能：将buf容器中的count个数据，写入到fd引用的文件中
       参数1：已经打开的文件的文件描述符
       参数2：要写入的数据的起始地址
       参数3：要写入数据的个数
       返回值：成功返回写入的字符个数，这个个数可能小于count，失败返回-1并置位错误码
```



```cpp

#include<myhead.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
#include<unistd.h>
int main(int argc,const char* argv[])
{
    //1、定义文件描述符，对于文件IO而言，句柄就是文件描述符
    int fd=-1;
    if((fd=open("./tt.txt",O_CREAT|O_WRONLY|0644))==-1){
        perror("open error");
        return -1;
    }
    printf("open success fd = %d\n",fd);//fd=3，因为012已经被分配
    char wbuf[128]="hello world";
    write(fd,wbuf,strlen(wbuf));

    close(fd);
    //再次以只读的形式打开文件，此时参数2中不需要使用O_CREAT，
    //那么第三个参数也不需要了
    if((fd=open("./tt.txt",O_RDONLY))==-1){
        perror("open error");
        return -1;
    }
    printf("open success fd = %d\n",fd);//fd=3，因为012已经被分配

    char rbuf[5] ="";
    //从文件中读取数据放入rbuf中
    int res = read(fd,rbuf,sizeof(rbuf));
    write(1,rbuf,res);//向1号文件描述符写入数据，之前读取多少，现在写入多少
    close(fd);
    return 0;
}
```

# 六、关于光标的操作：lseek
```cpp
       #include <sys/types.h>
       #include <unistd.h>
       off_t lseek(int fd, off_t offset, int whence);
       功能：移动光标的位置，并返回光标现在所在的位置
       参数1：文件描述符
       参数2：偏移量
           >0:表示从指定位置向后偏移n个字节
           <0:表示从指定位置向前偏移n个字节
           =0:在指定位置处不偏移
       参数3：偏移的起始位置
           SEEK_SET：文件起始位置
           SEEK_CUR：文件指针当前位置
           SEEK_END：文件结束位置
       返回值：光标现在所在的位置
       注意： lseek = fseek+ftell
```

```cpp

#include<myhead.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
#include<unistd.h>
int main(int argc,const char* argv[])
{
    //1、定义文件描述符，对于文件IO而言，句柄就是文件描述符
    int fd=-1;
    //以读写的形式创建文件，如果文件不存在则创建文件，如果文件存在则清空文件
    //如果创建文件时没有给权限，则该文件的权限是随机权限
    //如果创建文件时，给定了文件的权限，则文件最终的权限是 给定的 mode&~umask

    if((fd=open("./tt.txt",O_CREAT|O_RDWR|O_TRUNC,0644))==-1){
        perror("open error");
        return -1;
    }
    printf("open success fd = %d\n",fd);//fd=3，因为012已经被分配
    char wbuf[128]="hello world";
    write(fd,wbuf,strlen(wbuf));
    //此时光标位于末尾
    lseek(fd,-5,SEEK_END);////从文件结束位置向前偏移5个字节
    

    char rbuf[5] ="";
    //从文件中读取数据放入rbuf中
    int res = read(fd,rbuf,sizeof(rbuf));
    write(1,rbuf,res);//向1号文件描述符写入数据，之前读取多少，现在写入多少
    close(fd);
    return 0;
}
```

# 七、练习：使用IO完成对图像的操作
+ 将一个bmp格式的图片信息读取出来，并将该图片的相关内容进行修改。
+ <font style="color:rgb(51,51,51);">关于bmp图像格式可以参考：</font><font style="color:rgb(65,131,196);">https://upimg.baike.so.com/doc/5386615-5623077.html</font>
+ <font style="color:rgb(51,51,51);">如何将windows下的文件，复制到linux系统的Ubuntu下</font>

<details class="lake-collapse"><summary id="ub82e2cc6"><span class="ne-text" style="font-size: 16px">步骤</span></summary><p id="u4c9668e1" class="ne-p"><span class="ne-text" style="font-size: 16px">打开cmd 【win+r，在里面输入】</span></p><p id="u07670258" class="ne-p"><span class="ne-text" style="font-size: 16px">scp 要拷贝的路径  l</span><span class="ne-text" style="color: rgb(51,51,51); font-size: 16px">inux下的用户名@ip地址:linux下存放的地址路径</span></p><p id="u0fd0a448" class="ne-p"><img src="https://cdn.nlark.com/yuque/0/2025/png/40383045/1743842138613-9e0f932e-0dcc-4987-9028-63704a533843.png" width="1038.6666666666667" id="u3106ec33" class="ne-image"></p><ul class="ne-ul"><li id="u8437f6dc" data-lake-index-type="0"><span class="ne-text">把png改成了bmp</span></li></ul><p id="u19f1b87e" class="ne-p"><img src="https://cdn.nlark.com/yuque/0/2025/png/40383045/1743842246445-2599367d-8d5c-4c4e-b266-8076799414bb.png" width="701.3333333333334" id="uc2a17d89" class="ne-image"></p></details>
```cpp

```

# 八、补充知识点
如果使用自定义文件 myhead，这个文件是在当前文件目录下。如果有一天，他不在当前文件目录下，我们如何使用到它？

```cpp
cd /usr/include/  #切换到根目录下的user/include/里
这个目录下能看见所有的头文件
将myhead.h文件，移动到根目录下的usr目录中的include目录下
sudo mv myhead.h /usr/include
可以直接创建在 /usr/include里面
sudo vi /usr/include/myhead.h
```

```cpp
#include "myhead.h"

int main(int argc, char const *argv[])
{
    //分别输出 标准输入，标准输出，标准出错的文件描述符
    printf("stdin->_fileno=%d\n",stdin->_fileno);
    printf("stdout->_fileno=%d\n",stdout->_fileno);
    printf("stderr->_fileno=%d\n",stderr->_fileno);
}

```

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743820348208-a882bf42-8342-40fb-b8d2-f312b9d2fb54.png)

---

ctrl+shift+p后输入Sinippets: Configure Snippets 

进入后点击cpp.json 

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743820916271-8386e23b-91a2-432c-898e-56a497f12529.png)

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743820956846-2fc7007a-298e-4a8e-8bf4-7cff95ae3180.png)

```cpp
"Simple C++ Program": {
	 	"prefix": "main",
	 	"body": [
	 		"#include<myhead.h>",
			"int main(int argc,const char* argv[])",
			"{",
			"    $1",
			"    return 0",
			"}"
	 	],
	 	"description": "Simple C++ Program template"
	}
```

+ **<font style="color:#DF2A3F;">第7行和第8行只能用空格来隔开，不能用tab隔开</font>**。

![](https://cdn.nlark.com/yuque/0/2025/png/40383045/1743821083521-7262c398-6209-4b6b-a4f5-507715c3569f.png)

