description: C++23 の新しい言語機能と標準ライブラリ機能の解説

# C++23 の新機能

C++23 の規格ドラフトに入ることが決まった機能を随時追加していきます。

### `std::size_t` 型の整数リテラルのためのサフィックスを追加 [(P0330)](http://wg21.link/p0330)
これまで `std::size_t` 型の値をリテラルで表現する方法がなく、次のような不便の原因になっていました。

```C++
#include <vector>
#include <algorithm>

int main()
{
	std::vector<int> v{ 0, 1, 2, 3 };

	// コンパイルエラー (64-bit 環境): i と s が異なる型
	for (auto i = 0u, s = v.size(); i < s; ++i)
	{

	}

    // OK
    for (auto i = size_t(0), s = v.size(); i < s; ++i)
	{

	}

	size_t s2 = std::min(1u, v.size()); // コンパイルエラー (64-bit 環境): 引数の型が不一致

    size_t s3 = std::min<size_t>(1u, v.size()); // OK
}
```

C++23 では `std::size_t` 型の値を表すリテラルのサフィックス `z` および `Z` が追加され、`123z` は符号付きの `std::size_t` に相当する型 (`std::ptrdiff_­t`), `123uz` が `std::size_t` 型と見なされます。 

```C++
#include <vector>
#include <algorithm>

int main()
{
	std::vector<int> v{ 0, 1, 2, 3 };

	// OK
	for (auto i = 0uz, s = v.size(); i < s; ++i)
	{

	}

	size_t s2 = std::min(1uz, v.size()); // OK
}
```

