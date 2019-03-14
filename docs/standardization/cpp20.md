description: C++20 の新しい言語機能と標準ライブラリ機能の解説

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


### 符号付き整数型の負数表現を 2 の補数と規定 [(P1236R1)](http://wg21.link/p1236r1)
ほぼすべての現代的なコンピュータで、符号付き整数型の負数は 2 の補数で表現されます。しかし、C++ では負数の表現方法について規格で定めていなかったため、（現実的ではありませんが）2 の補数以外で負数表現を実装する余地も残されていました。C++20 からは 2 の補数のみを許可するよう規格文言が修正されました。  
近年のアーキテクチャで 2 の補数以外を使う例は、1 の補数を使う Unisys 2200 があります。また過去には符号と絶対値で表現するアーキテクチャもありました。これらはモダンな C++ を開発環境として使用しないので、この規格変更による影響は無いと考えられます。

#### （参考）符号付き 8-bit 整数の 2 進表現
| 10 進表現 | 2 の補数    | 1 の補数    | 符号と絶対値   |
|-------|----------|----------|----------|
| 127   | 01111111 | 01111111 | 01111111 |
| 126   | 01111110 | 01111110 | 01111110 |
| 125   | 01111101 | 01111101 | 01111101 |
| 2     | 00000010 | 00000010 | 00000010 |
| 1     | 00000001 | 00000001 | 00000001 |
| 0     | 00000000 | 00000000 | 00000000 |
| -0    |          | 11111111 | 10000000 |
| -1    | 11111111 | 11111110 | 10000001 |
| -2    | 11111110 | 11111101 | 10000010 |
| -126  | 10000010 | 10000001 | 11111110 |
| -127  | 10000001 | 10000000 | 11111111 |
| -128  | 10000000 |          |          |


### メンバのオブジェクトが空のクラスの場合にメモリ消費を 0 にできる `[[no_unique_address]]` 属性を追加 [(P0840R2)](http://wg21.link/p0840r2)
アロケータなどをクラスのメンバとして保持するとき、それがステートレスな空のクラスであってもオブジェクトのアドレスを一意に用意しないといけないため、サイズを 0 にできずメモリ消費が無駄に増えてしまう問題がありました。
```C++
#include <iostream>

struct Empty {};

struct X
{
	int i;
	Empty e;
};

int main()
{
	std::cout << sizeof(X) << '\n'; // 4 より大きい
}
```
これを回避するために「空の基底クラスは最適化によってサイズ 0 にしてよい」という仕様を利用した Empty Base Optimization (EBO) というテクニックがあり、標準ライブラリでも　`std::unique_ptr` や `std::shared_ptr`, `std::vector` などに使われています。
```C++
#include <iostream>

struct Empty {};

struct X : public Empty
{
	int i;
};

int main()
{
	std::cout << sizeof(X) << '\n'; // 4
}
```
しかし、これらのクラスで継承による EBO を実装するとコードが複雑になるという欠点がありました。C++20 ではメンバの宣言に `[[no_unique_address]]` 属性を付けることで、継承を使わなくてもコンパイラが EBO と同じような最適化をできるようになり、従来の継承による EBO を使っていたコードを、より単純なコードに置き換えられます。
```C++
#include <iostream>

struct Empty {};

struct X
{
	int i;
	[[no_unique_address]] Empty e;
};

int main()
{
	std::cout << sizeof(X) << '\n'; // 4
}
```


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