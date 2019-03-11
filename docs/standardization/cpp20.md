# C++20 の新機能

## 言語機能

### ビットフィールドにデフォルトの初期値を設定可能に [(P0683R1)](http://wg21.link/p0683r1)
```C++
#include <iostream>

enum class Terrain
{
    Open, Forest, Hill, Mountain, Desert, Tundra, River, Ocean
};

struct Tile
{
    unsigned int height : 4 = 1; // デフォルト値を 1 に
    Terrain terrain : 3 = Terrain::Open; // デフォルト値を明示的に Terrain::Open に
    bool passable : 1 = true; // デフォルト値を true に
};

int main()
{
    std::cout << std::boolalpha;
    
    Tile tile1;
    std::cout << tile1.height << ", " << static_cast<int>(tile1.terrain) << ", " << tile1.passable << '\n';
    
    Tile tile2{ 15, Terrain::Mountain, false };
    std::cout << tile2.height << ", " << static_cast<int>(tile2.terrain) << ", " << tile2.passable << '\n';
}
```
```
1, 0, true
15, 3, false
```

### メンバポインタ演算子の仕様を一貫性のために修正 [(P0704R1)](http://wg21.link/p0704r1)

C++17 までのメンバポインタ演算子 `.*` は「右辺値オブジェクトから、左辺値参照修飾されたメンバ関数ポインタに使うことは不適格」という規格文面になっていました。そのため、同じ意味をもつ次の 2 つのプログラムで後者だけ不適格とされ、一貫性がありませんでした。
```C++
#include <iostream>
#include <string>

struct Text
{
	std::string m_data;

	const std::string& get() const &
	{
		return m_data;
	}
};

int main()
{
	std::cout << Text{ "Hello" }.get() << '\n'; // OK

	std::cout << (Text{ "Hello" }.*&Text::get)() << '\n'; // C++17 までは不適格、C++20 から OK
}
```
C++20 では「右辺値オブジェクトから、左辺値参照修飾された "非 const" メンバ関数ポインタに使うことは不適格」と文面を修正し、後者もコンパイルできるようになります。


## 標準ライブラリ

### 文字列の先頭や末尾が、ある文字列と一致するか判定 [(P0457R2)](https://wg21.link/P0457R2)
`std::basic_string` と `std::basic_string_view` に、`starts_with()` と `ends_with()` メンバ関数が追加されます。
```C++
#include <iostream>
#include <string_view>

constexpr bool HasPNGExtension(std::string_view filePath)
{
    // 文字列が ".png" で終わるなら true, それ以外は false を返す
    return filePath.ends_with(".png");
}

int main()
{
    std::cout << std::boolalpha;
    std::cout << HasPNGExtension("picture.png") << '\n';
    std::cout << HasPNGExtension("photo.jpg") << '\n';
    std::cout << HasPNGExtension("music.mp3") << '\n';
}
```
```
true
false
false
```


### `operator>>(basic_istream&, charT*)` の第二引数を `charT(&)[N]` に変更して安全に [(P0487R1)](https://wg21.link/P0487R1)

C++17 までの `operator>>(basic_istream&, charT*)` は、関数にバッファのサイズが渡されないため、次のようなプログラムでバッファオーバーフローへの対策が必要でした。
```C++
#include <iostream>
#include <iomanip>

int main()
{
	char buffer[4];
	
	// std::cin >> buffer; // 危険: バッファオーバーフローの可能性
	
	std::cin >> std::setw(4) >> buffer; // OK: バッファオーバーフロー対策
	
	std::cout << buffer;
}
```
C++20 では引数を次のように変更し、関数がバッファオーバーフローの対策を実装するようになります。
```C++
// C++17 まで
template<class charT, class traits>
basic_istream<charT, traits>& operator>>(basic_istream<charT, traits>& in, charT* s);

// C++20 から
template<class charT, class traits, size_t N>
basic_istream<charT, traits>& operator>>(basic_istream<charT, traits>& in, charT (&s)[N]);
```
```C++
#include <iostream>
#include <iomanip>

int main()
{
	char buffer[4];
	
	std::cin >> buffer; // OK: C++20 ではバッファオーバーフローを防げる
	
	std::cout << buffer;
}
```
この変更に伴い、C++17 までは有効だった次のようなプログラムが、C++20 からコンパイルエラーになります。
```C++
#include <iostream>
#include <iomanip>

int main()
{
    char* p = new char[100];
    std::cin >> std::setw(100) >> p; // C++20 からはコンパイルエラー
    std::cout << p;
}
```

### 戻り値の無視が不具合をもたらす関数に `[[nodiscard]]` を付与 [(P0600R1)](https://wg21.link/P0600R1)
C++17 で導入された `[[nodiscard]]` 属性を標準ライブラリで活用するようになります。C++20 では付与基準を「戻り値の無視がトラブルやメモリリークなどの重大なエラーを引き起こす C++ の関数」とし、`async()`, `launder()`, `allocate()`, `empty()`, `operator new()` が対象となっています。
```C++
#include <vector>
#include <future>

int main()
{
	std::vector<int> v = { 10, 20, 30 };

	v.empty(); // C++20 では警告

	std::async(std::launch::async, [] { return 1; }); // C++20 では警告
}
```
MSVC の標準ライブラリでは Visual Studio 2017 15.6 以降、規格の範囲を超えてより多くの関数（[2,500 個以上](https://devblogs.microsoft.com/cppblog/c17-progress-in-vs-2017-15-5-and-15-6/)）に `[[nodiscard]]` 属性を使っています。その結果、[Chromium のソースに無意味な std::move() が見つかる](https://bugs.chromium.org/p/webrtc/issues/detail?id=8463)など、既存のコードベースのバグの発見に役立っています。


### `<array>` ヘッダのすべての関数が constexpr に [(P1023R0)](https://wg21.link/P1023R0), [(P1032R1)](https://wg21.link/P1032R1)

C++17 の `<array>` ヘッダでは、比較演算子、`swap()`, `fill()` 以外のすべての関数が constexpr でした。C++20 ではさらに、array の比較演算の実装に使われている `std::equal()` と `std::lexicographical_compare()` が [constexpr になった (P0202R3)](https://wg21.link/P0202R3) ことにともない、array の比較演算子を constexpr とし、また `swap()` と `fill()` についても constexpr にすることを決め、array ヘッダのすべての関数が constexpr で提供されます。