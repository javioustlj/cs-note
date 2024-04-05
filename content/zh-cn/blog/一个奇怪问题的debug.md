---
title: 一个奇怪问题的debug之旅
linkTitle: 一个奇怪问题的debug之旅
date: 2024-04-05
description: >
    介绍了debug一次奇怪问题的过程！
---


## 问题

公司的守护进程的优雅关闭，是通过捕获 `SIGTERM` 实现的：
1. 捕获 `SIGTERM`
2. 退出处理（释放资源等等）
3. 最后调用 `pthread_exit` 退出

出现的问题是：优雅关闭时，进程又收到了 `SIGABRT` ，导致进程又被异常杀死了，最后进程并没有优雅的关闭。

当然在公司定位这个问题的过程中，都是通过日志和 gdb 工具来进行的。

现在我们模拟一下解决这个问题的过程，为了方便起见，就使用打印和 gdb 来进行debug

模拟的问题代码如下：

```cpp
#include <iostream>
#include <pthread.h>
#include <signal.h>

void printSigInfo(int signo, siginfo_t* info, void* context)
{
    std::cout << "Signal Info:" << std::endl;
    std::cout << "  si_signo: " << info->si_signo << " - Signal number" << std::endl;
    std::cout << "  si_code: " << info->si_code << " - Signal code" << std::endl;
    std::cout << "  si_errno: " << info->si_errno << " - Errno value associated with signal" << std::endl;
    if (info->si_code > 0 && info->si_code < NSIG) { // 可能需要针对具体情况进行检查
        std::cout << "  si_pid: " << info->si_pid << " - Sending process ID" << std::endl;
        std::cout << "  si_uid: " << info->si_uid << " - Real user ID of sending process" << std::endl;
        if (info->si_addr) {
            std::cout << "  si_addr: " << static_cast<void*>(info->si_addr) << " - Faulting address" << std::endl;
        }
    }
}

void LinuxSigHandler(int signo, siginfo_t* info, void* context)
{
    switch (signo) {
        case SIGABRT:
            std::cout << "Caught signal SIGABRT (" << signo << ")." << std::endl;
            printSigInfo (signo, info, context);
            break;
        case SIGTERM:
            std::cout << "Caught signal SIGTERM (" << signo << ")." << std::endl;
            printSigInfo (signo, info, context);
            pthread_exit(0);
            break;

        default:
            std::cout << "Caught signal " << signo << std::endl;
            break;
    }

    // 重新引发信号
    raise(signo);
}

void InitSignalHandlers(void)
{
    struct sigaction action;
    action.sa_handler = NULL;
    action.sa_sigaction = LinuxSigHandler;
    sigemptyset (&action.sa_mask);
    action.sa_flags = (typeof(action.sa_flags))(SA_RESTART | SA_SIGINFO | SA_RESETHAND);
    sigaction (SIGSEGV, &action, NULL);
    sigaction (SIGILL, &action, NULL);
    sigaction (SIGFPE, &action, NULL);
    sigaction (SIGBUS, &action, NULL);
    sigaction (SIGABRT, &action, NULL);
    sigaction (SIGTERM, &action, NULL);
    /* Ignored signals. */
    struct sigaction action_ignore;
    action_ignore.sa_handler = SIG_IGN;
    action_ignore.sa_flags = 0;
    sigemptyset (&action_ignore.sa_mask);
    sigaction (SIGPIPE, &action_ignore, NULL);
}

void main_thread()
{
    for (;;)
    {
        std::cout << "main_thread is running:" << std::endl;
        sleep(10);
    }
}

int main()
{
    InitSignalHandlers();
    try {
        main_thread();
    } catch (const std::exception& e) {
        std::cerr << "Caught exception: " << e.what() << std::endl;
    }
    catch (...) {
        std::cerr << "Caught unknown exception" << std::endl;
    }
    std::cerr << "process close" << std::endl;
    return 0;
}
```


我们先来观察一段这段代码，它有如下特点：

1. 它是一个守护进程，死循环在处理业务逻辑
2. 它调用了 `sigaction` 函数修改了与指定信号的相关联的处理动作
	1. 注意 `SIGTERM`： 收到这个信号，做打印并调用了 `pthread_exit`
	2. `SIGABRT`: 收到这个信号，做打印；
3. 当收到 `SIGTERM` 时，它应该有打印，并优雅的退出，不应该再有其他的打印。

我们如何测试呢？
1. 我们测试这个程序现在用 `kill -SIGTERM` 实现。公司的系统优雅的关闭就是通过 `systemd` 发送 `SIGTERM` 实现的。

现在我们来做一个测试吧！

我们来以这段代码做个测试，编译这段代码得到可执行文件 `abi`

```shell
g++ abi.cpp -o abi
```

在窗口 1 运行它

```shell
./abi
main_thread is running:
main_thread is running:
```

在窗口 2 里面执行：

```sh
ps -ef | grep abi
tlj       980622  976483  0 08:36 pts/0    00:00:00 ./abi
$ kill -SIGTERM 980622
```


观察窗口 1：

```sh
./abi
main_thread is running:
main_thread is running:
Caught signal SIGTERM (15).
Signal Info:
  si_signo: 15 - Signal number
  si_code: 0 - Signal code
  si_errno: 0 - Errno value associated with signal
Caught unknown exception
FATAL: exception not rethrown
Caught signal SIGABRT (6).
Signal Info:
  si_signo: 6 - Signal number
  si_code: -6 - Signal code
  si_errno: 0 - Errno value associated with signal
Aborted (core dumped)
```

## 分析

??? 怎么又收到了一个 `SIGABRT` ，还异常关闭了？

从打印可以看到

```sh
Caught unknown exception
```

这里捕获了一个异常，那么按照正常的逻辑，它应该继续往下执行 `std::cerr << "process close" << std::endl;` 才对，结果它没有往下执行，到这里就收到了 `SIGABRT`.

十分怪异！

那我们先来看一下 `core dump` 看看发生了什么？

先启用coredump
```sh
ulimit -c unlimited
echo "/var/crash/core-%e-%p-%t" > /proc/sys/kernel/core_pattern
```

用 gdb 看一下 core 的情况：

```sh
root@ubuntu:/mnt/work/cs-note-src/src/cpp/debug# gdb ./abi core_abi_2262_1712044154
GNU gdb (Ubuntu 12.1-0ubuntu1~22.04) 12.1
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./abi...
[New LWP 2262]
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Core was generated by `./abi'.
Program terminated with signal SIGABRT, Aborted.
#0  __pthread_kill_implementation (no_tid=0, signo=6, threadid=139802159551424) at ./nptl/pthread_kill.c:44
44      ./nptl/pthread_kill.c: No such file or directory.
(gdb)
(gdb) bt
#0  __pthread_kill_implementation (no_tid=0, signo=6, threadid=139802159551424) at ./nptl/pthread_kill.c:44
#1  __pthread_kill_internal (signo=6, threadid=139802159551424) at ./nptl/pthread_kill.c:78
#2  __GI___pthread_kill (threadid=139802159551424, signo=signo@entry=6) at ./nptl/pthread_kill.c:89
#3  0x00007f263a21b476 in __GI_raise (sig=sig@entry=6) at ../sysdeps/posix/raise.c:26
#4  0x00007f263a2017f3 in __GI_abort () at ./stdlib/abort.c:79
#5  0x00007f263a2623dc in __libc_message (action=do_abort, fmt=0x7f263a3b479c "%s", fmt=0x7f263a3b479c "%s", action=do_abort)
    at ../sysdeps/posix/libc_fatal.c:155
#6  0x00007f263a2626f0 in __GI___libc_fatal (message=<optimized out>) at ../sysdeps/posix/libc_fatal.c:164
#7  0x00007f263a2763f6 in unwind_cleanup (reason=<optimized out>, exc=<optimized out>) at ./nptl/unwind.c:114
#8  0x0000562cea54094f in main () at new_abi.cpp:85
(gdb)

```

从这里看出来似乎就是从 main 函数发生了错误，剩下的调用栈都是库函数了。

到这里就没有思路了，main 哪里写的有问题？

在 `catch(...)` 的时候出错了，我们去掉，重新测试，还真没有了异常！

```sh
root@ubuntu:/mnt/work/cs-note-src/src/cpp/debug# ./abi
main_thread is running:
Caught signal SIGTERM (15).
Signal Info:
  si_signo: 15 - Signal number
  si_code: 0 - Signal code
  si_errno: 0 - Errno value associated with signal
```

那我们基本就知道了，是 `catch(...)` 带来的这个问题。那么究竟是怎么带来的呢？

那我们就要了解 C++ stack unwinding

### Stack unwinding 栈展开

**堆栈展开**是在运行时从函数调用堆栈中删除函数条目的过程。局部对象以与构建它们的相反顺序被销毁。

堆栈展开通常与[异常处理](https://www.geeksforgeeks.org/exception-handling-c/)有关。在 C++ 中，当发生异常时，将线性搜索函数调用堆栈以查找异常处理程序，并且从函数调用堆栈中删除具有异常处理程序的函数之前的所有条目。因此，如果在同一函数（抛出异常的位置）中未处理异常，则异常处理涉及堆栈展开。基本上，堆栈展开是为运行时构造的所有自动对象调用析构函数（每当引发异常时）的过程。


```cpp
#include <iostream>
#include <string>
#include <memory>

using namespace std;

class A
{
public:
    A (int a) : a(a) { cout << "A::A(): " << a << endl; }
    virtual ~A() { cout << "A::~A(): " << a << endl; }
    void disp() { cout <<"A disp" << endl; }
private:
    int a;
};

class B : public A
{
public:
    B (int a, const string &b) : A(a), b(b) { cout << "B::B(): " << b << endl; }
    ~B() { cout << "B::~B(): " << b << endl; }
    void disp() { cout <<"B disp" << endl; }
private:
    int a;
    string b;
};

void f1()
{
    cout << "f1() Start " << endl;
    B b1(1, "stack object");
    A *P = new B(2, "heap object");
    shared_ptr<A> p2 = make_shared<B>(3, "shared ptr object");  
    throw 100;
    cout << "f1() End " << endl;
}

void f2()
{
    cout << "f2() Start " << endl;
    f1();
    cout << "f2() End " << endl;
}

void f3()
{
    cout << "f3() Start " << endl;
    try {
        f2();
    }
    catch (int i) {
        cout << "Caught Exception: " << i << endl;
    }
    cout << "f3() End" << endl;
}

int main()
{
    f3();
    return 0;
}
```

```sh
f3() Start
f2() Start
f1() Start
A::A(): 1
B::B(): stack object
A::A(): 2
B::B(): heap object
A::A(): 3
B::B(): shared ptr object
B::~B(): shared ptr object
A::~A(): 3
B::~B(): stack object
A::~A(): 1
Caught Exception: 100
f3() End

```

当异常发生的时候，如果没有异常处理程序，这些局部变量一个一个去销毁的。需要注意的是 `p2` ，当异常发生时，没有发生析构，这说明，这里会有**资源泄露**，所以当在实际项目中，需要注意到这一点。

现在我们知道了异常处理的过程，那么我们的测试程序究竟捕获了什么异常呢？为什么又导致了 `SIGABRT` 呢？

从调用栈，我们看到 `main` 函数调用的是 `nptl/unwind.c` 中的 `unwind_cleanup` 函数，咱们先看看这个函数到底是怎么回事？


```cpp
// https://github.com/lattera/glibc/blob/master/nptl/unwind.c
static void
unwind_cleanup (_Unwind_Reason_Code reason, struct _Unwind_Exception *exc)
{
  /* When we get here a C++ catch block didn't rethrow the object.  We
     cannot handle this case and therefore abort.  */
  __libc_fatal ("FATAL: exception not rethrown\n");
}
```

看这里的注释

> “当程序运行到这里，那就是 C++ catch 块没有重新抛出对象。我们无法处理这种情况，所以触发 abort”

那么就可以明白了，我们 catch 了一个异常，但是没有重新抛出，所以 glibc 出发了 abort。

那么我们 catch 的这个异常到底是什么呢？从我们的程序可以看出，我们只是调用了 `pthread_exit`, `pthread` 本身是应该只是 C 库，并不是 C++库，它不应该抛出异常啊。

这和 `glibc` 和 `gcc` 的实现相关！

`pthread_exit` 或者 `pthread_cancel` 都会抛出一个 `___forced_unwind` 异常，用于在线程退出期间展开堆栈 (stack unwinding). 当程序捕获了这个异常，需要务必重新抛出它，以便系统可以做完整的 `stack unwinding`。

如果捕获了这个异常，那么栈展开就没办法继续进行下去了，所以 glibc 就会调用系统调用 abort 函数来进行 `SIGABRT` 异常处理。

## 修改 并测试
知道了真正原因，那么修改我们原有的程序，让它优雅的关闭吧

```cpp
int main()
{
    InitSignalHandlers();
    try {
        main_thread();
    } catch (const std::exception& e) {
        std::cerr << "Caught exception: " << e.what() << std::endl;
    }
	catch (abi::_forced_unwind&) {
		std::cerr << "Caught forced_unwid error and retrow it" << std::endl;
		throw;
	}
    catch (...) {
        std::cerr << "Caught unknown exception" << std::endl;
    }
    std::cerr << "process close" << std::endl;
    return 0;
}
```

测试一下，成功了，程序可以优雅关闭了！

```sh
main_thread is running:
main_thread is running:
Caught signal SIGTERM (15).
Signal Info:
  si_signo: 15 - Signal number
  si_code: 0 - Signal code
  si_errno: 0 - Errno value associated with signal
Caught forced_unwid error and retrow it

```


## 总结:
1. 异常处理程序，要千万注意资源泄露；
2. 异常处理程序的 stack unwinding 的过程是怎么样的。
3. 通过 gdb 和 core dump 来进行 debug。


Reference:

1. https://stackoverflow.com/questions/11452546/why-does-pthread-exit-throw-something-caught-by-ellipsis
2. https://cxx-abi-dev.codesourcery.narkive.com/VlHf6T8W/gcc-unwind-abi-change-for-forced-unwind
3. https://www.geeksforgeeks.org/stack-unwinding-in-c/
4. https://blog.csdn.net/Jqivin/article/details/121908435
5. https://copyprogramming.com/howto/pthread-exit-null-not-working
