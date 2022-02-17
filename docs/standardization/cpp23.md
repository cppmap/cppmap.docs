description: C++23 の新しい言語機能と標準ライブラリ機能の解説

# C++23 の新機能

C++23 の規格ドラフトに入ることが決まった新機能・仕様変更を随時追加していきます。

## 言語機能

### `std::size_t` 型の整数リテラルのためのサフィックスを追加 [(P0330)](http://wg21.link/P0330)
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

	size_t s2 = std::max(1u, v.size()); // コンパイルエラー (64-bit 環境): 引数の型が不一致

    size_t s3 = std::max<size_t>(1u, v.size()); // OK
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

	size_t s2 = std::max(1uz, v.size()); // OK
}
```


## 標準ライブラリ

<!-- P0881R7, P2301R1 -->

### C 言語との atomics の互換を目的とした標準ライブラリヘッダ `<stdatomic.h>` を追加 [(P0943R6)](http://wg21.link/P0943R6)
C++ atomics の非ジェネリックな部分 (`atomic_char` や `atomic_ulong` など) は C 言語からも使えるように設計されていましたが、C 言語がジェネリック用として独自に `_Atomic(T)` 型指定子を追加したほか、C++ の規格では C 言語のヘッダ `<stdatomic.h>` に関する言及がないことから、実際の相互運用にはいくらかの手間が必要でした。

```C++
#ifdef __cplusplus
	#include <atomic>	
	using std::atomic_int;	
	using std::memory_order;
	using std::memory_order_acquire;
	#define _Atomic(X) std::atomic<X>
	// ...
#else
	#include <stdatomic.h>
#endif

// C でも C++ でもコンパイル可能
int main(void)
{
	atomic_int a;
	memory_order b = memory_order_acquire;
	_Atomic(int) c;
    return 0;
}
```

C++23 では標準ライブラリに C 言語との atomics の互換のためのヘッダ `<stdatomic.h>` を追加し、インクルードするだけで共通のコードを書けるようにします。

```C++
#include <stdatomics.h>

// C でも C++ でもコンパイル可能
int main(void)
{
	atomic_int a;
	memory_order b = memory_order_acquire;
	_Atomic(int) c;
    return 0;
}
```


### 型が scoped enum であるかを調べる `std::is_scoped_enum<T>` trait [(P1048R1)](http://wg21.link/P1048R1)
C++11 で scoped enum (`enum class` / `enum struct`) が導入されましたが、同時に導入された type trait `std::is_enum<T>` は、unscoped enum (`enum`) 型と scoped enum 型のどちらにも `true` を示し、両者を区別できませんでした。

C++23 では型が scoped enum であるかを調べる `std::is_scoped_enum<T>` trait が追加され、両者を区別できるようになります。

```C++
#include <iostream>
#include <type_traits>

enum UnscopedEnum {};
enum class ScopedEnum {};

int main()
{
	std::cout << std::boolalpha;
	
	std::cout << std::is_enum_v<int> << '\n';			// false	
	std::cout << std::is_enum_v<UnscopedEnum> << '\n';	// true
	std::cout << std::is_enum_v<ScopedEnum> << '\n';	// true
	
	std::cout << std::is_scoped_enum_v<int> << '\n';			// false
	std::cout << std::is_scoped_enum_v<UnscopedEnum> << '\n';	// false
	std::cout << std::is_scoped_enum_v<ScopedEnum> << '\n';		// true
}
```

なお、Boost.TypeTraits にはすでに次のように実装されています。

```C++
namespace boost
{
	template <class T>
	struct is_scoped_enum
		: conjunction<is_enum<T>, negation<is_convertible<T, int>>>::type {};
}
```


### 文字列クラスに、指定した文字や文字列が含まれているかを返す `.contains()` メンバ関数 [(P1679R3)](https://wg21.link/P1679R3)
`std::string` や `std::string_view` にある文字や文字列が含まれているかを調べるには、次のように `.find()` を使った少し遠回りなコードを書く必要がありました。

```C++
#include <iostream>
#include <string>

int main()
{
	std::string s = "C++23";

	std::cout << std::boolalpha;

	// '4' が含まれているかを調べる
	std::cout << (s.find('4') != std::string::npos) << '\n'; // false

	// "++" が含まれているかを調べる
	std::cout << (s.find("++") != std::string::npos) << '\n'; // true
}
```

C++23 では `std::basic_string` と `std::basic_string_view` に、指定した文字や文字列が含まれるかを返す `.contains(basic_string_view)`, `.contains(charT)`, `.contains(const charT*)` メンバ関数が追加され、より簡潔に書けるようになりました。

```C++
#include <iostream>
#include <string>

int main()
{
	std::string s = "C++23";

	std::cout << std::boolalpha;

	// '4' が含まれているかを調べる
	std::cout << s.contains('4') << '\n'; // false

	// "++" が含まれているかを調べる
	std::cout << s.contains("++") << '\n'; // true
}
```

なお、指定した文字列から始まるかを調べる `.starts_with()`, 指定した文字列で終わるかを調べる `.ends_with()` メンバ関数が C++20 で追加されています。


### 列挙型の値を基底型に変換する `std::to_underlying()` [(P1682R3)](https://wg21.link/P1682R3)
これまで `enum class` を基底型の整数値へ適切に変換するには `static_cast<std::underlying_type_t<Enum>>(e)` と書く必要がありました。C++23 ではこれを `std::to_underlying(e)` と書けるようになりました。

```C++
#include <iostream>
#include <utility>

enum class State : char
{
	Open, Closed
};

enum class Terrain
{
	Open, Mountain, River, Ocean
};

int main()
{
	auto a = std::to_underlying(State::Open); // c は char

	auto b = std::to_underlying(Terrain::Mountain); // b は int
}
```


<!-- P2212R2 -->

<!-- P2162R2 -->

<!-- P2017R1 -->

<!-- P2259R1 -->

<!-- P0401R6 -->



### `std::stringstream` と異なり、内部バッファに `std::span` を用いる `std::spanstream` [(P0448R4)](https://wg21.link/P0448R4)
`std::stringstream` は内部のバッファを動的に確保するものでしたが、C++23 で追加された `std::spanstream` は `std::span` をバッファとして用いることができるようになりました。例えば、スタックに確保した固定サイズのバッファを割り当てることができます。

```C++
# include <iostream>
# include <string_view>
# include <spanstream>
# include <sstream>

int main()
{
	const char input[] = "11 22 33";

	// C++23
	{
		std::ispanstream is{ input }; // span としてバッファを渡す
		int a, b, c;
		is >> a >> b >> c;

		char buffer[30]{};
		std::ospanstream os{ buffer }; // span としてバッファを渡す
		os << (a * 100) << (b * 100) << (c * 100);
		std::cout << std::string_view{ os.span() } << '\n';
	}

	// (比較) std::stringstream
	{
		std::istringstream is{ input }; // std::string が作られる
		int a, b, c;
		is >> a >> b >> c;

		std::ostringstream os; // 内部バッファは動的に確保される
		os << (a * 100) << (b * 100) << (c * 100);
		std::cout << os.str() << '\n';
	}
}
```

<!-- P1132R8 -->

### ` type_info::operator==()` が constexpr に [(P1328R1)](https://wg21.link/P1328R1)
C++20 で `typeid` が constexpr の文脈で使えるようになりましたが、戻り値の `std::type_info` には constexpr のメンバ関数が無く実用性がありませんでした。C++23 では `std::type_info::operator==` が constexpr になりました。

```C++
#include <iostream>
#include <typeinfo>

struct IShape
{
	virtual ~IShape() = default;
};

struct Circle : IShape
{
	double x, y, r;
};

int main()
{
	if constexpr (typeid(IShape) != typeid(Circle))
	{
		std::cout << "typeid(IShape) != typeid(Circle))\n";
	}
}
```


### `std::stack` と `std::queue` に、イテレータペアをとるコンストラクタオーバーロードを追加 [(P1425R4)](https://wg21.link/P1425R4)
`std::stack` と `std::queue` には、イテレータペアをとるコンストラクタが無く、他のコンテナやコンテナアダプタと一貫性が無かったため、C++23 で追加されました。これは、C++23 で追加された `ranges::to` の内部実装が複雑になるのを防ぐことにも貢献します。

```C++
#include <vector>
#include <stack>
#include <queue>

int main()
{
	std::vector<int> v = { 10, 20, 30, 40 };
	std::stack<int> s(v.begin(), v.end());
	std::queue<int> q(v.begin(), v.end());
}
```


<!-- P1518R2 -->


<!-- P1659R3 -->


<!-- P2166R1 -->


<!-- P2136R3 -->


<!-- P1989R2 -->


<!-- P1951R1 -->


<!-- P2186R2 -->


