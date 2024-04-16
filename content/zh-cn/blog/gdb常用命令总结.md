---
title: gdb常用命令总结
linkTitle: gdb常用命令总结
date: 2024-04-05
description: >
    记录使用gdb调试时常用的命令
---

## 1. cheat sheet

| 显示类命令       | 缩写      | 命令说明         |
| :---------- | ------- | ------------ |
| info        | i       | 查看断点 / 线程等信息 |
| print       | p       | 打印变量或寄存器值    |
| display     | display | 自动显示命令       |
| whatis      | whatis  | 查看变量类型       |
| ptype       | ptype   | 查看变量类型       |
| list        | l       | 显示源码         |
| disassemble | dis     | 查看汇编代码       |
| backtrace   | bt      | 查看当前线程的调用堆栈  |
| help        | help    | 帮助命令         |

| 控制类型的命令  | 缩写     | 命令说明                     |
| -------- | ------ | ------------------------ |
| run      | r      | 运行一个待调试的程序               |
| continue | c      | 让暂停的程序继续运行               |
| next     | n      | 运行到下一行                   |
| step     | s      | 单步执行，遇到函数会进入             |
| until    | u      | 运行到指定行停下来                |
| finish   | fi     | 结束当前调用函数，回到上一层调用函数处      |
| return   | return | 结束当前调用函数并返回指定值，到上一层函数调用处 |
| jump     | j      | 将当前程序执行流跳转到指定行或地址        |

| 断点监视点   | 缩写      | 命令说明                 |
| ------- | ------- | -------------------- |
| break   | b       | 添加断点                 |
| tbreak  | tb      | 添加临时断点               |
| delete  | d       | 删除断点                 |
| enable  | enable  | 启用某个断点               |
| disable | disable | 禁用某个断点               |
| watch   | watch   | 监视某一个变量或内存地址的值是否发生变化 |

| 名称        | 缩写        | 命令说明           |
| --------- | --------- | -------------- |
| frame     | f         | 切换到当前调用线程的指定堆栈 |
| thread    | thread    | 切换到指定线程        |
| set args  | set args  | 设置程序启动命令行参数    |
| show args | show args | 查看设置的命令行参数     |

### 1.1 常用调试方式
1. 直接调试目标程序：
2. 附加进程 id：gdb attach pid
3. 调试 core 文件：gdb filename corename

调试目标程序
```
gdb ./hello_world
```

调试一个正在运行的程序

```sh
# 启动时直接attach到一个进程
gdb -p PID
```
```sh
# 在gdb环境中 attach一个进程
（gdb） attach PID
```

调试 core 文件：
```
gdb ./program_name corename
```

### 1.2 常用命令

退出命令：**q（quit）或者 Ctr + d**
启动程序：**r (run)**
继续运行：**c (continue）**
查看调用栈：**bt (backtrace）**
进入调用栈：**f （frame）堆栈编号**
单步执行：**n（next）**，遇到函数直接跳过，不进入函数内部
单步执行：**s（step）**，会进入函数内部
退出当前函数：**finish**，继续执行完当前函数，正常退出
返回当前函数：**return**，从当前位置，从当前函数退出，剩下的代码不执行了，可以指定函数的返回值；
指定位置停下来：**until**，和 **break** 类似
查看某段代码的汇编指令：**disassemble**
帮助命令：**help**

### 1.3 断点

#### 1.3.1 设置断点

程序运行到某一行
```
b test.cpp 100
```

程序运行到某一个函数
```
b namespace::func
```

其他文件设置
```
b a.cpp:441
```

其他文件的某一个函数
```
b a.cpp:func
```

设置条件断点
```
b if i = 8
```

#### 1.3.2 查看断点

```
info b
```

#### 1.3.3 删除断点

删除第几个断点
```
delete N
```

删除第几行的断点

```
clear N
```

#### 1.3.4使能断点

使能第 N 个断点
```
enable N
```

不使能第 N 个断点

```
disable N
```

### 1.4 监视点

单个变量
```
watch i
```
指针变量，监视的是指针变量本身，
```
watch p
```
指针所指向的变量，监视指针所值的内容：
```
watch *p
```
数组，可以监视整个数组的内容
```
watch a
```

注意：当监视的变量变化时会自动停下来
	当监视的变量脱离了作用域，也就失效了；

### 1.5 打印

调用函数
```
p func()
```

调试过程中修改变量的值
```
print var=value
```

计算表达式并打印
```
print a+b+c
```

输出当前对象的各成员变量的值
```
print *this
```

查看变量类型
```
whatis var
```

查看变量类型，可以查看复合数据类型，会打印出该类型的成员变量
```
ptype val
```

#### 1.5.1 打印字符串

打印 `std::string`

打印内容
```
p str
p str.c_str()
```
打印长度
```
p str.length()
p str.size()
```
字符串太长
```
set print elements 0
```

#### 1.5.2 打印 vector 和 map


### 1.6 `list` 命令

输出上一次list命令显示的代码后面的代码，如果是第一次执行list命令，则会显示当前正在执行代码位置附近的代码；
```
list
```

上一次 list 命令显示的代码前面的代码
```
list -
```

显示当前代码文件第 **Linenumber** 行附近的代码；
```
list Linenumber
```

显示 **FileName** 文件第 **LineNo** 行附近的代码
```
list FileName:Linenumber
```

显示当前文件的 **FunctionName** 函数附近的代码
```
list FunctionName
```

显示 **FileName** 文件的 **FunctionName** 函数附件的代码
```
list FileName:FunctionName
```

其中**from**和**to**是具体的代码位置，显示这之间的代码
```
list from,to
```

### 1.7 `jump` 命令

1. 中间跳过的代码是不会执行的；
2. 跳到的位置后如果没有断点，那么GDB会自动继续往后执行；

跳转到第几行
```
jump line_number
```
跳转往下 10行
```
jump + 10
```
跳转到具体的内存地址
```
jump *0X12345678
```

### 1.8 display 命令

基本用法
```bash
display expression
```

1. **跟踪变量值：** 如果您正在调试一个C程序，并且想观察变量`foo`的值随程序执行的变化，可以这样设置：
    ```bash
    (gdb) display foo
    ```
2. **显示内存地址内容：** 若要跟踪特定内存地址（如`0x12345678`）的内容：
    ```bash
    (gdb) display *0x12345678
    ```
    
3. **跟踪表达式结果：** 如果您关心的是某个计算结果或复杂表达式的值，比如变量`a`与`b`之和：
    ```bash
    (gdb) display a + b
    ```
4. **格式化输出：** 可以使用`/fmt`选项来指定显示值的格式，类似于`printf`函数的格式化字符串。例如，以十六进制显示整数：
    ```bash
    (gdb) display/x foo
    ```
    或以浮点数的科学记数法显示：
    ```bash
    (gdb) display/g foo
    ```
    查看GDB文档或使用`help format`命令获取更多关于格式化选项的信息。
5. **查看已设置的display项：** 如果您设置了多个`display`命令，可以使用`info display`命令列出所有当前生效的自动显示项及其编号：
    ```bash
    (gdb) info display
    ```
6. **取消自动显示：** 要取消之前设置的某个`display`项，使用`undisplay`命令并指定其编号：
    ```bash
    (gdb) undisplay 1
    ```
    上述命令将取消编号为1的`display`设置。
    
7. **临时禁用所有自动显示：** 如果想暂时关闭所有自动显示，而不取消它们的设置，可以使用`disable display`命令：
    ```bash
    (gdb) disable display
    ```
    
    后续可以通过`enable display`命令重新启用所有自动显示。
    
8. **查看即将执行的汇编指令：** 在调试汇编程序时，使用`display/i $pc`命令可以显示每次程序暂停时即将执行的汇编指令：
    ```bash
    (gdb) display/i $pc
    ```
    `$pc`是GDB的内部变量，代表当前程序计数器的值，即下一条要执行的指令地址。


## 2 调试分析 core 文件

1. 用 gdb 打开 core 文件
2. 查看堆栈信息
3. 进入某个栈：f N，f 是 frame 的缩写，N 是栈号，如0、1、2、3...
4. 进入栈之后，查看变量，print
```sh
gdb ./program core
bt
where
f 3
```

## 3 多线程调试

### 3.1 常用的命令

查看所有线程信息
```
info thread
```
切换线程
```
thread thread_number
```
查看所有线程的栈信息
```
thread apply all bt
```

在特定线程编号上打断点

```
break location thread thread_number
```

通用的命令, 可以用 all，作用禹全部线程
```
thread apply thread_number1 thread_number2 ... command
```

### 3.2 模式

- **all-stop mode**，全停模式，当程序由于任何原因在 GDB 下停止时，不止当前的线程停止，所有的执行线程都停止。这样允许你检查程序的整体状态，包括线程切换，不用担心当下会有什么改变。  
- **non-stop mode**，不停模式，调试器（如VS2008和老版本的GDB）往往只支持 **all-stop** 模式，但在某些场景中，我们可能需要调试个别的线程，并且不想在调试过程中影响其他线程的运行，这样可以把GDB的调式模式由 **all-stop** 改成 **non-stop**，**7.0** 版本的GDB引入了 **non-stop** 模式。在 **non-stop** 模式下 **continue、next、step** 命令只针对当前线程。  
- **record mode**，记录模式；记录模式允许 GDB 记录程序的执行过程，包括线程的运行、变量的修改等，并将这些信息保存到记录文件中。这个功能可以帮助您分析程序的运行过程，并且可以在之后的时间点回放这些记录，从而帮助您定位问题或理解程序的执行流程。
- **replay mode**，回放模式；回放模式允许您回放之前记录的执行过程。在这个模式下，您可以重现程序之前的执行状态，以便更深入地分析和调试程序。这种模式在查找程序运行中的难以复现的问题时非常有用。

non-stop 模式
```sh
gdb -exec-run-mode non-stop your_program
```

记录模式
```sh
gdb -record record_file_name your_program
```

回放模式
```sh
gdb -replay record_file_name
```
### 3.3 设置线程锁

默认的调试模式：**一个线程暂停运行，其他线程也随即暂停；一个线程启动运行，其他线程也随即启动**

但在一些场景中，我们希望只让特定线程运行，其他线程都维持在暂停状态，即要防止**线程切换**。

- **set scheduler-locking on**，锁定线程，只有当前或指定线程可以运行；专注于一个调试一个线程。
- **set scheduler-locking off**，在单步调试当前线程时，其他线程不受影响，继续并发执行。
- **set scheduler-locking step**，当单步执行某一线程时，其他线程不会执行，同时保证在调试过程中当前线程不会发生改变。但如果在该模式下执行 **continue、until、finish** 命令，则其他线程也会执行；
- **show scheduler-locking**，查看线程锁定状态；

## 4 技巧

和 shell 一样，有一些简便操作：

1. 上下方向键可以查看最近的命令
2. 回车，执行上一次执行的命令，单步调试非常有用

Reference:

1. https://zhuanlan.zhihu.com/p/297925056
2. https://www.zhihu.com/question/65306462

