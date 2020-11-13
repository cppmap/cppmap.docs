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

### 文字列クラスに、指定した文字列が含まれるかを返す `.contains()` メンバ関数を追加 [(P1679)](http://wg21.link/P1679)
Java, C#, Rust の文字列クラスは、指定した文字列を含むかのメソッドを持っていますが、C++ の `std::string` には同等のメンバ関数がなく、次のようなコードを書く必要がありました。

```C++
#include <iostream>
#include <string>

int main()
{
	const std::string s = "I like C++23";

	if (s.find("C++") != std::string::npos) // 文字列に "C++" が含まれるかを調べる
	{
		std::cout << "found!\n";
	}
}
```

C++23 では `std::basic_string` と `std::basic_string_view` に、指定した文字や文字列が含まれるかを返す `.contains(basic_string_view)`, `.contains(charT)`, `.contains(const charT*)` メンバ関数が追加され、より短く書けるようになります。

```C++
#include <iostream>
#include <string>

int main()
{
	const std::string s = "I like C++23";

	std::cout << std::boolalpha;
	std::cout << s.contains('+') << '\n';	// true
	std::cout << s.contains('-') << '\n';	// false
	std::cout << s.contains("like") << '\n';	// true
	std::cout << s.contains("C++11") << '\n';	// false
	std::cout << s.contains(s) << '\n';	// true
}
```

なお、指定した文字列から始まるかを調べる `.starts_with()`, 指定した文字列で終わるかを調べる `.ends_with()` メンバ関数は C++20 で追加されています。


### 型が scoped enum であるかを調べる `std::is_scoped_enum<T>` trait [(P1048)](http://wg21.link/P1048)
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
