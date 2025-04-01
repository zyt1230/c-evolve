```cpp
gcc -g 文件 -o 可执行程序
gdb ./可执行程序
r
l m,n
l func
b
info breakpoint
delete 编号
watchpoint
shell
set logging on
set variable 变量=值
n
c
s
p
display 变量      将变量加入常显示（每次停下来都显示它的值）。
undisplay 编号   取消指定编号变量的常显示。
「disable 编号」：禁用指定编号的断点。
「enable 编号」：启用指定编号的断点。
```
