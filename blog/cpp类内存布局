---
title: cpp类内存布局
linkTitle: cpp类内存布局
date: 2024-04-16
description: >
    Linux OS 64 位 用gdb看一下cpp类内存布局
---

## 没有虚函数的情况

```cpp
#include <iostream>

class A
{
public:
    A() : c(0)
    {
        std::cout << "A::A()" << std::endl;
        func2();
    };
    void func2() { std::cout <<"A::func2()" << std::endl; };
private:
    int c;
};

int main(void)
{
    A a1;
    A a2;
    
    return 0;
}
```

```sh
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
Reading symbols from ./a.out...
(gdb) b 22
Breakpoint 1 at 0x11fc: file basic.cpp, line 22.
(gdb) run
Starting program: /mnt/work/cs-note-src/src/cpp/memory/a.out
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
A::A()
A::func2()
A::A()
A::func2()

Breakpoint 1, main () at basic.cpp:22
22          return 0;
(gdb) info vtbl a1
This object does not have a virtual function table
(gdb)
This object does not have a virtual function table
(gdb) p a
$1 = {i = {0, 1045149306}, x = 1.2904777690891933e-08, d = 1.2904777690891933e-08}
(gdb) p a1
$2 = {c = 0}
(gdb) p a2
$3 = {c = 0}
(gdb) p &a1
$4 = (A *) 0x7fffffffe250
(gdb) p &a2
$5 = (A *) 0x7fffffffe254
(gdb) p typeid(a1)
could not find typeinfo symbol for 'A'
(gdb) info address A::func2
Symbol "A::func2()" is a function at address 0x5555555552da.
(gdb) info symbo A::func2
A::func2() in section .text of /mnt/work/cs-note-src/src/cpp/memory/a.out
(gdb)
```

## 存在虚函数的情况


```

```sh
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
Reading symbols from ./virtual...
(gdb) b 24
Breakpoint 1 at 0x11fc: file virtual.cpp, line 24.
(gdb) run
Starting program: /mnt/work/cs-note-src/src/cpp/memory/virtual
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
A::A()
A::func2()
A::A()
A::func2()

Breakpoint 1, main () at virtual.cpp:24
24          return 0;
(gdb) p a1
$1 = {_vptr.A = 0x555555557d68 <vtable for A+16>, c = 0}
(gdb) p &a1
$2 = (A *) 0x7fffffffe230
(gdb) x /2xg 0x7fffffffe230
0x7fffffffe230: 0x0000555555557d68      0x0000000000000000
(gdb) info vtbl a1
vtable for 'A' @ 0x555555557d68 (subobject @ 0x7fffffffe230):
[0]: 0x5555555552ea <A::virtual_func1()>
[1]: 0x555555555366 <A::virtual_func3()>
(gdb) p typeid(a1)
$3 = {_vptr.type_info = 0x7ffff7fa2fa0 <vtable for __cxxabiv1::__class_type_info+16>, __name = 0x55555555603c <typeinfo name for A> "1A"}
(gdb) p sizeof(typeid(a1))
$4 = 16
(gdb) p &typeid(a1)
$5 = (gdb_gnu_v3_type_info *) 0x555555557d78 <typeinfo for A>
(gdb) x /4xg 0x555555557d68
0x555555557d68 <_ZTV1A+16>:     0x00005555555552ea      0x0000555555555366
0x555555557d78 <_ZTI1A>:        0x00007ffff7fa2fa0      0x000055555555603c
0x555555557d88: 0x0000000000000001      0x0000000000000169
0x555555557d98: 0x0000000000000001      0x0000000000000178
```

```sh
0x555555557d80        0x000055555555603c
0x555555557d78        0x00007ffff7fa2fa0
0x555555557d70        0x0000555555555366
0x555555557d68        0x00005555555552ea
```


## 虚函数继承

```cpp
#include <iostream>

class A
{
public:
    A() : c(0)
    {
        std::cout << "A::A()" << std::endl;
        func2();
    };
    virtual void virtual_func1() { std::cout << "A::virtual_func1()" << std::endl;};
    void func2() { std::cout <<"A::func2()" << std::endl; };
    virtual void virtual_func3() { std::cout << "A::virtual_func3()" << std::endl;};
private:
    int c;
};

class B : public A
{
public:
    B() : a(0), b(0)
    {
        std::cout << "B::B()" << std::endl;
        func2();
    }
    B(int a, int b) : a(a), b(b)
    {
        std::cout << "B::B(int a, int b)" << std::endl;
        func2();
    }
    ~B() { std::cout << "B::~B()" << std::endl; };
    virtual void virtual_fun1() { std::cout <<"B::virtual_fun1()" << std::endl; };
    void func2() { std::cout <<"B::func2()" << std::endl; };
    virtual void virtual_func3() { std::cout << "B::virtual_func3()" << std::endl;};
private:
    int a;
    int b;
};

int main(void)

{
    A a1;
    A a2;
    B b1;
    B b2;

    return 0;

}
```

```sh

(gdb) b 48
Breakpoint 1 at 0x1235: file ./inheritance.cpp, line 48.
(gdb) run
Starting program: /mnt/work/cs-note-src/src/cpp/memory/inheritance
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
A::A()
A::func2()
A::A()
A::func2()
A::A()
A::func2()
B::B()
B::func2()
A::A()
A::func2()
B::B()
B::func2()

Breakpoint 1, main () at ./inheritance.cpp:48
48          return 0;
(gdb) p b1
$1 = {<A> = {_vptr.A = 0x555555557d10 <vtable for B+16>, c = 0}, a = 0, b = 0}
(gdb) p b2
$2 = {<A> = {_vptr.A = 0x555555557d10 <vtable for B+16>, c = 0}, a = 0, b = 0}
(gdb) p &b1
$3 = (B *) 0x7fffffffe210
(gdb) p &b2
$4 = (B *) 0x7fffffffe230
(gdb) p typeid(b1)
$5 = {_vptr.type_info = 0x7ffff7fa3c30 <vtable for __cxxabiv1::__si_class_type_info+16>, __name = 0x55555555607b <typeinfo name for B> "1B"}
(gdb) p typeid(b2)
$6 = {_vptr.type_info = 0x7ffff7fa3c30 <vtable for __cxxabiv1::__si_class_type_info+16>, __name = 0x55555555607b <typeinfo name for B> "1B"}
(gdb) info vtbr b1
Undefined info command: "vtbr b1".  Try "help info".
(gdb) info vtbl b1
vtable for 'B' @ 0x555555557d10 (subobject @ 0x7fffffffe210):
[0]: 0x555555555362 <A::virtual_func1()>
[1]: 0x55555555555e <B::virtual_func3()>
[2]: 0x5555555554e2 <B::virtual_fun1()>
(gdb) p &typeid(b1)
$7 = (gdb_gnu_v3_type_info *) 0x555555557d48 <typeinfo for B>
(gdb) x /16xg 0x555555557d10
0x555555557d10 <_ZTV1B+16>:     0x0000555555555362      0x000055555555555e
0x555555557d20 <_ZTV1B+32>:     0x00005555555554e2      0x0000000000000000
0x555555557d30 <_ZTV1A+8>:      0x0000555555557d60      0x0000555555555362
0x555555557d40 <_ZTV1A+24>:     0x00005555555553de      0x00007ffff7fa3c30
0x555555557d50 <_ZTI1B+8>:      0x000055555555607b      0x0000555555557d60
0x555555557d60 <_ZTI1A>:        0x00007ffff7fa2fa0      0x000055555555607e
0x555555557d70: 0x0000000000000001      0x00000000000001b6
0x555555557d80: 0x0000000000000001      0x00000000000001c5

```

```sh
0x555555557d68       0x000055555555607e  --> typeid(a)  __name = 1A
0x555555557d60       0x00007ffff7fa2fa0  --> typeid(a) _vptr.type_info
0x555555557d58       0x0000555555557d60
0x555555557d50       0x000055555555607b  --> typeid(b)  __name = 1B
0x555555557d48       0x00007ffff7fa3c30  --> typeid(b) _vptr.type_info
0x555555557d40       0x00005555555553de  --> A::virtual_func3()
0x555555557d38       0x0000555555555362  --> A::virtual_func1()
0x555555557d30       0x0000555555557d60
0x555555557d28       0x0000000000000000
0x555555557d20       0x00005555555554e2  --> B::virtual_fun1()
0x555555557d18       0x000055555555555e  --> B::virtual_func3()
0x555555557d10       0x0000555555555362  --> A::virtual_func1()
```


