---
title: "cpp17"
linkTitle: "cpp17"
weight: 2
draft: false
description: >
    C++ 17 引入的新特性
---

1. 嵌套命名空间； Nested Namespaces 
2. If 和 switch 中的变量声明；Variable declaration in if and switch；
3. If constexpr 语句
4. 结构化绑定
5. 折叠表达式
6. 枚举的直接列表初始化
7. 内联变量
8. `constexpr lambda` 表达式
9. 新增三个 `attributes`


### 1. Nested Namespaces 嵌套命名空间



C++17 之前

```cpp
namespace Game {
	namespace Graphics {
		namespace Physics {
		class 2D {
			..........
		};
		} // Physics
	} // Graphics
} // Game
```

C++17 之后

```cpp
namespace Game::Graphics::Physics {

	class 2D {
	..........
	};
}
```

### 2. Variable declaration in if and switch

C++17 之前

```cpp
std::vector<std::string> str;
const auto it = std::find(std::begin(str), std::end(str), "abc");
if (it != std::end(str))
{
	*it = "$$$";
}
```

C++17 之后

```cpp
std::vector<std::string> str;
if (const auto it = std::find(std::begin(str), std::end(str), "abc"); it != std::end(str) )
{
	*it = "$$$";
}
```

### 3. If constexpr statement


 **if constexpr** is evaluated at _compile time_, like `constexpr` function.

```cpp
template <typename T>
auto length(T const &value)
{
	if constexpr(is_integral<T>::value)
	{
		return value;
	}
	else
	{
		return value.length();
	}
}
```

### 4. Structure Bindings 结构化绑定

C++17 之前

```cpp
map<string, string> fav_lang{
	{ "John", "Java" },
	{ "Alex", "C++" },
	{ "Peter", "Python" }
};

auto result = fav_lang.insert({ "Henry", "Golang" });
bool success = result.second;
auto process = result.first;

for (const auto& pair : fav_lang) {
	const string& name = pair.first;
	const string& lang = pair.second;
	cout << name << ":" << lang << endl;
}
```

C++17 之后
```cpp
map<string, string> fav_lang{ 
	{ "John", "Java" }, 
	{ "Alex", "C++" }, 
	{ "Peter", "Python" } 
}; 

auto[process, success] 
	= fav_lang.insert({ "Henry", 
						"Golang" }); 
 
for (const auto & [ name, lang ] : fav_lang) { 
	cout << name << ":"
		 << lang << endl; 
} 
```


### 5. Folding Expressions

C++17 之前
```cpp
auto CPP11_sum()
{
	return 0;
}

template<typename T1, typename... T>
auto C11_sum(T1 s, T... ts)
{
	return s + CPP11_sum(ts...);
}

```

C++17 之后
```cpp
template<typename... Args>
auto sum(Args... args)
{
	return (args + ... + 0);
}

template <typename... Args>
auto sum2(Args... args)
{
	return (args + ...);
}
```

### 6. Direct list initialization of enums


C++17 之前

```cpp
enum class byte : unsigned char {};
byte b = byte(0); // OK
byte c = byte(-1); // ERROR，-1 超出了 unsigned char 的范围
byte d = byte(1); // OK
byte e = byte(256); // ERROR，256 超出了 unsigned char 的范围
```

CPP 17 之后
```cpp
enum byte : unsigned char {};   
byte b {0}; // OK   
byte c {-1}; // ERROR   -1 超出了 unsigned char 的范围
byte d = byte{1}; // OK   
byte e = byte{256}; // ERROR 256 超出了 unsigned char 的范围
```

### 7. Inline variables  内联变量

全局变量：
```cpp
// header.h
inline int variable = 42;

// a.cpp
#include "header.h"

// b.cpp
#include "header.h"
```

类的 inline 变量

```cpp


// header.h
class myClass { 
public: 
	inline static int var1;
	inline static int var2 = 10; 
}; 
inline int myClass::var1 = 5;

```


### 8. `constexpr lambda` 表达式

```cpp
constexpr auto lamb = [] (int n) { return n * n; };
static_assert(lamb(3) == 9, "a");
```


### 9. 新增三个 `attributes`

`[[fallthrough]]`
```cpp
void f(int n)
{
    switch (n)
    {
        case 1:
        case 2:
            g();
            [[fallthrough]]; // 告诉编译器，没有break是正常的，
        case 3: 
            h();
        default:
    }
}
```

`[[nodiscard]]`
```cpp
[[nodiscard]] int func();
void f()
{
	func(); // 会有告警，不要忽略返回值；
}
```

`[[maybe_unused]]`

```cpp
[[maybe_unused]] void f1(){}
void f2(){}
void f3()
{
	int x = 0;
	[[maybe_unused]] int y = 1; // y没用到，可以消除没用到的告警
	return x + 1;
}
```
