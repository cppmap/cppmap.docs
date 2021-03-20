description: 正しいように見えて、コンパイルエラーになる C++ コードの紹介

# なぜかコンパイルできない

正しいように見えて、コンパイルエラーになるコードの紹介です。

## [添字とラムダ式](https://wandbox.org/permlink/SVY9fqkBjRUSUNJQ)
```C++
#include <iostream>

int main()
{
	int a[3] = { 10, 20, 30 };

	std::cout << a[[](){ return 1; }()] << '\n';
}
```

## [使えない変数](https://wandbox.org/permlink/HJHm8EKL6Ma73jyR)
```C++
#include <memory>

int main()
{
	std::shared_ptr<int> p();

	p.reset();
}
```

## [ポインタ型のデフォルト引数](https://wandbox.org/permlink/RiDJ2HEPG9zL0KH8)
```C++
void Func(int*=nullptr)
{

}

int main()
{
	Func();
}
```

## [関数のオーバーロード](https://wandbox.org/permlink/dTWsrIqVeuJRifdK)
```C++
#include <iostream>

using T = int&;

void f(T&)
{
	std::cout << "A\n";
}

void f(const T&)
{
	std::cout << "B\n";
}

int main()
{
	int x;
	f(x);
}
```

## [0xe](https://wandbox.org/permlink/aYCRa87L7rhq5Ydt)
```C++
int main()
{
    return 0xe-0xe;
}
```
