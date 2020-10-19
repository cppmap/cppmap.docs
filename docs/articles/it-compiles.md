description: C++ でコンパイルできる、不思議な見た目のコードの紹介

# なぜかコンパイルできる

C++ の規格上コンパイルできる、不思議な見た目のコードの紹介です。

## ソースコードに URL
```C++
int main()
{
	https://cppmap.github.io
	return 0;
}
```

## main 関数 try ブロック
```C++
int main()
try
{

}
catch (...)
{
	return -1;
}
```

## 添字式の入れ替え
```C++
#include <iostream>

int main()
{
	int a[3] = { 10, 20, 30 };

	std::cout << 2[a] << '\n';

	for (int i = 0; i < 3; ++i)
	{
		std::cout << i[a] << '\n';
	}
}
```

## 括弧の連続
```C++
int main()
{
	{};
	[]{};
	[](){};
	[](){}();
}
```

## 空のプリプロセッサディレクティブ
```C++
#
#include <iostream>
#

int main()
{
	#
}
```

## `-->` 演算子
```C++
#include <iostream>

int main()
{
	int i = 10;

	while (i --> 0)
	{
		std::cout << i << '\n';
	}
}
```

## 同じ関数
```C++
#include <iostream>

using ll = long long;

void f(unsigned ll)
{
	std::cout << "A\n";
}

void f(unsigned long long)
{
	std::cout << "B\n";
}

int main()
{
	f(123ull);
}
```

## 戻り値が無いのに `[[nodiscard]]`
```C++
[[nodiscard]] void Func()
{

}

int main()
{
	Func(); // 警告は出ない
}
```

## 長い関数
```C++
struct C
{
	inline virtual volatile constexpr const unsigned long long int f() const noexcept final = delete;
};

int main()
{

}
```

## 名前空間エイリアスの再帰
```C++
namespace A
{
	namespace A = A;

	void f() {}
}

int main()
{
	A::A::A::A::A::A::f();
}
```

## `typedef` の場所 1
```C++
long unsigned typedef int long Uint64;

int main()
{
	Uint64 i = 123ull;
}
```

## `typedef` の場所 2
```C++
int main()
{
	if (typedef int Int; true)
	{
		Int a = 0;
	}

	switch (typedef int Int; 0)
	{
	case 0:
		Int a = 0;
		break;
	}
}
```
