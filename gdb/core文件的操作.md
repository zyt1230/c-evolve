```cpp
先处理
ulimit -a
ulimit -c unlimited
//ubuntu检查
sysctl kernel.core_pattern
echo "core.%e.%p" | sudo tee /proc/sys/kernel/core_pattern
```


