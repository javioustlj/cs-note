---
title: cpp程序性能优化技巧
linkTitle: cpp程序性能优化技巧
date: 2025-08-27
description: >
    cpp 程序性能优化技巧汇总
---


# 技巧一

利用RAII给一个函数计时

```cpp

struct Timer {
    std::string name;
    std::chrono::steady_clock::time_point start;
    Timer(const std::string& n) : name(n), start(std::chrono::steady_clock::now()) {}
    ~Timer() {
        auto end = std::chrono::steady_clock::now();
        auto duration = duration_cast<std::chrono::microseconds>(end - start).count();
        std::cout << name << " took " << duration << " microseconds\n";
    }
};

void f2()
{
    timer t1 ("f2");
    // some operation
}

void f1()
{
    f2();
}

```

程序中t1的计时包含了所有局部变量析构的时间，比直接在return前打点计时更准确。

另外当f1中对f2调用打点计时和f2中的计时差值就是系统开销了。
