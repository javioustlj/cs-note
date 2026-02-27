---
title: ncclIntruQueue
weight: 12
date: 2025-10-13
description:
    ncclIntruQueue
---

## 1. ncclIntruQueue简介

NCCL实现了一个侵入式队列的通用模板。Intru 即 (intrusive)，侵入式的。C++标准库中的队列都是非侵入式队列。

侵入式队列（Intrusive Queue）与非侵入式队列（Non-intrusive Queue）的核心区别在于元素是否自带链接指针：
- 侵入式队列：元素本身包含用于队列链接的指针（如next指针），队列操作直接通过元素自身的指针完成。
- 非侵入式队列：元素本身不包含链接信息，队列通过额外的 “包装节点”（如struct Node { T data; Node* next; }）来维护元素间的链接。


c语言写一个小例子，简单看一下侵入式队列和非侵入式队列的区别。

```cpp
struct Task {
    int id;
    struct Task* next;  // 侵入式指针
};
struct Task* queue_head = NULL;
```

```cpp
struct Task {
    int id;
};

struct Node {
    struct Task task;
    struct Node *next;
};
struct Node* queue_head = NULL;
```

一般来说，std 中的队列实现是一种标准实现，考虑了更通用的场景，比如是dequeue，而不是一个单向列表，会引入更多的开销，在高性能计算场景，或者嵌入式场景，可以实现一个更简单的队列。

NCCL实现侵入式队列代码如下：

```cpp
////////////////////////////////////////////////////////////////////////////////

template<typename T, T *T::*next>
struct ncclIntruQueue {
  T *head, *tail;
};

template<typename T, T *T::*next>
inline void ncclIntruQueueConstruct(ncclIntruQueue<T,next> *me) {
  me->head = nullptr;
  me->tail = nullptr;
}

template<typename T, T *T::*next>
inline bool ncclIntruQueueEmpty(ncclIntruQueue<T,next> *me) {
  return me->head == nullptr;
}

template<typename T, T *T::*next>
inline T* ncclIntruQueueHead(ncclIntruQueue<T,next> *me) {
  return me->head;
}

template<typename T, T *T::*next>
inline T* ncclIntruQueueTail(ncclIntruQueue<T,next> *me) {
  return me->tail;
}

template<typename T, T *T::*next>
inline void ncclIntruQueueEnqueue(ncclIntruQueue<T,next> *me, T *x) {
  x->*next = nullptr;
  (me->head ? me->tail->*next : me->head) = x;
  me->tail = x;
}

template<typename T, T *T::*next>
inline void ncclIntruQueueEnqueueFront(ncclIntruQueue<T,next> *me, T *x) {
  if (me->head == nullptr) me->tail = x;
  x->*next = me->head;
  me->head = x;
}

template<typename T, T *T::*next>
inline T* ncclIntruQueueDequeue(ncclIntruQueue<T,next> *me) {
  T *ans = me->head;
  me->head = ans->*next;
  if (me->head == nullptr) me->tail = nullptr;
  return ans;
}

template<typename T, T *T::*next>
inline bool ncclIntruQueueDelete(ncclIntruQueue<T,next> *me, T *x) {
  T *prev = nullptr;
  T *cur = me->head;
  bool found = false;

  while (cur) {
    if (cur == x) {
      found = true;
      break;
    }
    prev = cur;
    cur = cur->*next;
  }

  if (found) {
    if (prev == nullptr)
      me->head = cur->*next;
    else
      prev->*next = cur->*next;
    if (cur == me->tail)
      me->tail = prev;
  }
  return found;
}

template<typename T, T *T::*next>
inline T* ncclIntruQueueTryDequeue(ncclIntruQueue<T,next> *me) {
  T *ans = me->head;
  if (ans != nullptr) {
    me->head = ans->*next;
    if (me->head == nullptr) me->tail = nullptr;
  }
  return ans;
}

template<typename T, T *T::*next>
void ncclIntruQueueTransfer(ncclIntruQueue<T,next> *dst, ncclIntruQueue<T,next> *src) {
  (dst->tail ? dst->tail->next : dst->head) = src->head;
  if (src->tail) dst->tail = src->tail;
  src->head = nullptr;
  src->tail = nullptr;
}

////////////////////////////////////////////////////////////////////////////////
```

## 2. 说明

本质就是一个**单向列表实现的单向队列**，实现FIFO，非常简单。

注意模板语法

```cpp
// T *T::*next  
template<typename T, T *T::*next>;
```

从右往左读，或者按类型声明的规则理解：

| 部分                   | 解释                                                        |
| :--------------------- | :---------------------------------------------------------- |
| next                   | 变量名（这里是模板参数名）                                  |
| T::*                   | 表示这是一个“指向 T 类型对象的成员指针”                     |
| T *                    | 表示这个成员是指向 T 的指针类型的成员                       |
| 整体：T *T::*next	next | 是一个模板参数，它的类型是：指向类型为 T* 的 T 类成员的指针 |


```cpp
template<typename T, T *T::*next>
inline void ncclIntruQueueEnqueue(ncclIntruQueue<T,next> *me, T *x) {
  x->*next = nullptr;
  /* ... */
}
```

| 部分                | 说明                                                              |
| :------------------ | :---------------------------------------------------------------- |
| x                   | 是一个指向 T 对象的指针，比如 Task* x                             |
| next                | 是模板参数，类型为 T* T::*，表示“指向 T 的某个 T* 成员的指针”     |
| ->*                 | 是 C++ 的指向成员访问操作符，用于通过对象指针和成员指针来访问成员 |
| x->*next            | 表示：访问 x 所指向的对象的、由 next 指定的那个成员               |
| x->*next = nullptr; | 将该成员设置为 nullptr                                            |
