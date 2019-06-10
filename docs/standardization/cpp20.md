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


### 型名であることが明らかな文脈で `typename` を省略可能に [(P0634R3)](https://wg21.link/P0634R3)
C++17 で依存名が型である場合に `typename` を付けないのは、派生クラス定義時の基底クラスの指定と、初期化子リストでの基底クラスの指定のみでした（厳密にはこの 2 つには `typename` を付けられません）。C++20 では、型名しか使えないさらにいくつかの文脈で `typename` が省略可能になります。次のサンプルコードの左右タブで比較できます。

```C++ tab="C++17"
#include <vector>
#include <string>

template <class T, class Alloc = typename T::allocator_type>
struct S : T::value_type // 派生クラス定義時の基底クラスの指定
{
	using value_type = typename T::value_type;

	S()
        : T::value_type() {} // 初期化子リストでの基底クラスの指定

	typename T::size_type max_size() const;

	auto data()->typename T::pointer;

	auto min_size() const
	{
		return static_cast<typename T::size_type>(0);
	}
};

template <class T> typename T::size_type MaxSize();

int main()
{
	S<std::vector<std::string>> s;
}
```

```C++ tab="C++20"
#include <vector>
#include <string>

template <class T, class Alloc = T::allocator_type> // OK
struct S : T::value_type // 派生クラス定義時の基底クラスの指定
{
	using value_type = T::value_type; // OK

	S()
        : T::value_type() {} // 初期化子リストでの基底クラスの指定

	T::size_type max_size() const; // OK

	auto data()->T::pointer; // OK

	auto min_size() const
	{
		return static_cast<T::size_type>(0); // OK
	}
};

template <class T> T::size_type MaxSize(); // OK

int main()
{
    S<std::vector<std::string>> s;
}

```


### 定数式での仮想関数呼び出しが可能に [(P1064R0)](https://wg21.link/P1064)
コンパイル時に決定可能であれば、参照やポインタを通した仮想関数の呼び出しを `constexpr` にできるようになります。`constexpr` 修飾された仮想関数を非 `constexpr` 関数でオーバーライドすることや、その逆も可能です。

```C++
struct Cpp
{
	virtual int version() const = 0;
};

struct Cpp17 : Cpp
{
	constexpr int version() const override
	{
		return 17;
	}
};

struct Cpp20 : Cpp
{
	constexpr int version() const override
	{
		return 20;
	}
};

constexpr int GetVersion(const Cpp& a)
{
	return a.version();
}

int main()
{
	constexpr Cpp17 cpp17;
	constexpr Cpp20 cpp20;

	static_assert(GetVersion(cpp17) == 17);
	static_assert(GetVersion(cpp20) == 20);
}
```


### `type_id` と `dynamic_cast` が constexpr に [(P1327R1)](https://wg21.link/P1327)
`dynamic_cast` と `type_id` が、例外を投げるケースを除いて `constexpr` になります。

```C++
#include <typeinfo>

struct Cpp
{
	virtual int version() const = 0;
};

struct Cpp17 : Cpp
{
	constexpr int version() const override
	{
		return 17;
	}
};

struct Cpp20 : Cpp
{
	constexpr int version() const override
	{
		return 20;
	}
};

int main()
{
	constexpr static Cpp17 cpp17;
	constexpr const Cpp* pCpp = &cpp17;
	constexpr auto& cpptype = typeid(*pCpp);

	constexpr const Cpp& refCpp = cpp17;
	constexpr const Cpp17& redCpp2 = dynamic_cast<const Cpp17&>(refCpp);
}
```

次のように例外を投げるケースでは `constexpr` にできずコンパイルエラーになります。

```C++
#include <typeinfo>

struct Cpp
{
	virtual int version() const = 0;
};

struct Cpp17 : Cpp
{
	constexpr int version() const override
	{
		return 17;
	}
};

struct Cpp20 : Cpp
{
	constexpr int version() const override
	{
		return 20;
	}
};

int main()
{
	constexpr Cpp* pCpp = nullptr;
	constexpr auto& cpptype = typeid(*pCpp); //コンパイルエラー: 例外 std::bad_typeid を投げるため constexpr 不可

	constexpr static Cpp17 cpp17;
	constexpr const Cpp& refCpp = cpp17;
	constexpr const Cpp20& redCpp2 = dynamic_cast<const Cpp20&>(refCpp); // コンパイルエラー: 例外 std::bad_cast を投げるため constexpr 不可
}
```


### 定数式において共用体のアクティブメンバの切り替えが可能に [(P1330R0)](https://wg21.link/P1330)
共用体のアクティブメンバとは、最後に初期化または値を代入したメンバのことです。C++17 では共用体の初期化やアクティブメンバへのアクセスを定数式で行えましたが、アクティブメンバの切り替えはできませんでした。定数式でのアクティブメンバの切り替えが可能になると、共用体によって実装される `std::string` や `std::optional` などの標準ライブラリクラスのメンバ関数の `constexpr` 対応を拡充できます。非アクティブメンバへのアクセスは未定義動作なので、定数式で行うとコンパイルエラーになります。

```C++
#include <cstdint>

union Value
{
	float f;
	std::uint32_t i;
};

constexpr Value GetFloat(float x)
{
	return Value{ x }; // value.f がアクティブメンバ
}

constexpr Value GetUint(std::uint32_t x)
{
	Value value = GetFloat(0.0f); // value.f がアクティブメンバ
	value.i = x; // value.i がアクティブメンバに
	return value;
}

int main()
{
	static_assert(GetUint(123).i == 123);
}
```


### 定数式の文脈では `try-catch` を無視するように [(P1002R1)](https://wg21.link/P1002)
これまで `constexpr` 関数の中には `try-catch` ブロックを書くことができませんでした。しかし、`std::vector` 等のコンテナを `constexpr` 対応するにあたっては、この制限が障壁となるため、C++20 では `constexpr` 関数の中の `try-catch` は、定数式として評価されるときには無視するよう仕様が改められます。定数式の評価中に例外を投げるようであればコンパイルエラーになります。`std::vector` などを `constexpr` 対応させるための措置であり、将来の C++ におけるコンパイル時例外処理の実現を否定するものではありません。

```cpp
#include <cstdint> 
#include <iostream> 
#include <exception> 

constexpr std::uint32_t AddU8(std::uint32_t a, std::uint32_t b)
{
	if ((a + b) >= 256)
	{
		throw std::exception{};
	}

	return a + b;
}

constexpr std::uint32_t DoubleU8(std::uint32_t n)
{
	try
	{
		return AddU8(n, n);
	}
	catch (const std::exception& except)
	{
		return 0;
	}
}

int main()
{
	static_assert(DoubleU8(123) == 246); // OK: 例外を投げずに定数式として評価可能

	//static_assert(DoubleU8(200) > 0); // コンパイルエラー: 定数式として評価される constexpr 関数内で例外を投げるため

	std::cout << "result: " << DoubleU8(200) << '\n'; // OK: 実行時に評価される関数で例外が発生する
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


### `<chrono>` ヘッダの `zero()`, `min()`, `max()` 関数が noexcept に [(P0972R0)](https://wg21.link/P0972R0)
`std::chrono::duration_values`, `std::chrono::duration`, `std::chrono::time_point` などの `zero()`, `min()`, `max()` 関数に noexcept が付きます。


### `pointer_traits` が constexpr に [(P1006R1)](https://wg21.link/P1006R1) 
`std::vector` を constexpr にするのに必要なため、`std::pointer_traits::pointer_to()` 関数が constrexpr になります。


### ポインタのアライメントを最適化ヒントとしてコンパイラに伝える `assume_aligned()` 関数 [(P1007R3)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1007r3.pdf)
データのアドレスが 16 バイトなどのサイズにアライメントされている場合、コンパイラが SIMD を使った最適なコードを生成できる可能性があります。あるポインタの指すデータがアライメントされていることをコンパイラに伝える方法として、GCC や Clang では `__builtin_assume_aligned()` や `__attribute__((assume_aligned(alignment)))`, ICC では `__assume_aligned()` などの独自拡張がありますが、標準化された方法はありませんでした。C++20 では、これらの差異を吸収する次のような関数テンプレートが提供されます。
```C++
template <size_t N, class T>
[[nodiscard]] constexpr T* assume_aligned(T* ptr);
```
実際には次のように使います。
```C++
void Multiply(float* x, size_t size, float factor)
{
	float* ax = std::assume_aligned<64>(x); // x が 64 バイトアライメントであることを伝える
	
	for (size_t i = 0; i < size; ++i) // ループが適宜最適化される
	{
		ax[i] *= factor;
	}
}
```


### スマートポインタの作成時に値をデフォルト初期化する make 関数を追加 [(P1020R1)](https://wg21.link/P1020R1)
実行時性能のために、`float` や `unsigned char` など組み込み型の配列の値をデフォルト初期化させたい（ゼロ初期化しない）ケースがあります。しかし、`make_unique` や `make_shared`, `allocate_shared` でスマートポインタを作成した場合には値初期化が実行されます。C++20 では、値初期化をせずにデフォルト初期化でスマートポインタを作成する関数 `make_unique_default_init`, `make_shared_default_init`, `allocate_shared_default_init` が追加されました。
```C++
#include <iostream>
#include <memory>

// 未初期化の変数を使う実験的なコード
int main()
{
	float v[4]; // デフォルト初期化

	for (int i = 0; i < 4; ++i)
	{
		std::cout << v[i] << '\n';
	}

	auto pv = std::make_unique<float[]>(4); // 値初期化 (0 初期化) 

	for (int i = 0; i < 4; ++i)
	{
		std::cout << pv[i] << '\n';
	}

	auto pd = std::make_unique_default_init<float[]>(4); // デフォルト初期化

	for (int i = 0; i < 4; ++i)
	{
		std::cout << pd[i] << '\n';
	}
}
```
出力例
```
2.20325e-38
4.11052e+32
1.3013e-45
2.48626e-38
0
0
0
0
2.30415e-38
2.51341e-38
4.63281e+30
2.32703e+17
```


### 非順序連想コンテナのルックアップ操作で、`key_type` と比較可能な型を変換せずに使えるように [(P0919R3)](http://wg21.link/P0919r3)
C++17 までの `unorderd_map` や `unordered_set` など非順序連想コンテナでは、`find()`, `count()`, `equal_range()` などルックアップを行うメンバ関数は引数に `key_type` をとり、例えば次のようなケースで `std::string` 型の一時オブジェクトが作成されて非効率でした。

```C++
#include <string>
#include <unordered_map>

int main()
{
	std::unordered_map<std::string, int> table = { /* ... */ };

	auto it = table.find("abc"); // std::string 型の一時オブジェクトが作成される
}
```

C++20 では、非順序連想コンテナのテンプレートパラメータ `Hash` が `transparent_key_equal` タグを持つときに、`key_type` 以外の型を引数にとるメンバ関数テンプレートのオーバーロードが使用可能になり、一時オブジェクトの作成を回避できるようになります。

```C++
#include <string>
#include <string_view>
#include <unordered_map>

struct string_hash
{
	using transparent_key_equal = std::equal_to<>;  // KeyEqual to use
	using hash_type = std::hash<std::string_view>;  // helper local type
	size_t operator()(std::string_view txt) const { return hash_type{}(txt); }
	size_t operator()(const std::string& txt) const { return hash_type{}(txt); }
	size_t operator()(const char* txt) const { return hash_type{}(txt); }
};

int main()
{
	using namespace std::literals;

	std::unordered_map<std::string, int, string_hash> table = { /* ... */ };

	auto it1 = table.find("abc"); // std::string 型の一時オブジェクトは作成されない

	auto it2 = table.find("abc"sv); // std::string 型の一時オブジェクトは作成されない
}
```


### 非順序連想コンテナのルックアップ操作に、計算済みハッシュ値を渡せるように [(P0920R2)](https://wg21.link/P0920R2)
同じ種類の非順序連想コンテナを複数扱い、同じキーを用いたルックアップ操作をそれらのコンテナに対して行うようなケースでは、キーのハッシュ値の計算を毎回行うのは冗長です。C++20 ではメンバ関数 `find()`, `count()`, `contains()`, `equal_range()` に、計算済みのハッシュ値を渡せるオーバーロードが追加されます。
```C++
#include <iostream>
#include <string>
#include <array>
#include <unordered_map>

int main()
{
	std::array<std::unordered_map<std::string, int>, 10> maps = { /*...*/ };

	const std::string key = "key";

	const auto hash = maps.front().hash_function()(key);

	for (auto& map : maps)
	{
		auto it = map.find(key, hash);

		// ...
	}
}
```


### 2 つの値の中点を計算する `std::midpoint()` 関数 [(P0811R3)](https://wg21.link/P0811R3)
2 つの値 `a`, `b` の中点を計算する際に、単純な `(a + b) / 2` という式ではオーバーフローを起こす可能性があります。C++20 で追加される `std::midpoint()` 関数では、整数に対して
```C++
constexpr Integer midpoint(Integer a, Integer b) noexcept
{
	using U = make_unsigned_t<Integer>;
	return a>b ? a-(U(a)-b)/2 : a+(U(b)-a)/2;
}
```
のような実装が使われ、オーバーフローを回避できます。`(a + b)` が奇数になるケースの結果は `a` の方向に丸められます。  
浮動小数点数に対しては次のような実装が使われます。
```C++
Float midpoint(Float a, Float b)
{
	return isnormal(a) && isnormal(b) ? a/2+b/2 : (a+b)/2;
}
```
```C++
#include <iostream>
#include <numeric>

int main()
{
	std::cout << (2'000'000'000 + 1'000'000'000) / 2 << '\n'; // オーバーフロー

	std::cout << std::midpoint(2'000'000'000, 1'000'000'000) << '\n'; // 1500000000

	std::cout << std::midpoint(1, 4) << '\n'; // 2
	
	std::cout << std::midpoint(4, 1) << '\n'; // 3
}
```


### 2 つの値の線形補間を計算する `std::lerp()` 関数 [(P0811R3)](https://wg21.link/P0811R3)
2 点 `a`, `b` の間をパラメータ `t` によって線形補間する関数が提供されます。計算結果 `r` は `a + t * (b - a)` によって求められますが、実装により `isfinite(a) && isfinite(b)` のとき

- `lerp(a, b, 0) == a && lerp(a, b, 1) == b`
- `0 <= t && t <= 1` のとき `isfinite(r)`
- `isfinite(t) && a == b` のとき `r == a`
- `isfinite(t) || !isnan(t) && (b - a) != 0` のとき `!isnan(r)`

また、`cmp(lerp(a,b,t2), lerp(a,b,t1)) * cmp(t2,t1) * cmp(b,a) >= 0` (cmp は -1, 0, 1 を返す三方比較関数とする)  
であることが保証されます。
```C++
#include <iostream>
#include <numeric>

int main()
{
	std::cout << std::lerp(0.0, 10.0, 0.0) << '\n'; // 0

	std::cout << std::lerp(0.0, 10.0, 0.3) << '\n'; // 3

	std::cout << std::lerp(0.0, 10.0, 1.0) << '\n'; // 10

	std::cout << std::lerp(0.0, 10.0, 1.2) << '\n'; // 12
}
```


### 実装固有の情報をまとめる `<version>` ヘッダを追加 [(P0754R2)](https://wg21.link/P0754R2)
`__cpp_lib_byte`, `__cpp_lib_void_t` のような標準ライブラリの機能テストマクロ、その他ライブラリのバージョンや実装固有の情報をまとめる目的の `<version>` ヘッダが追加されました。
例えば C++20 以前の MSVC の標準ライブラリでは、`<yvals_core.h>` という独自ヘッダに標準ライブラリの機能テストマクロがまとめられていましたが、C++20 以降ではあらゆる実装において、`<version>` ヘッダを見ることで、こうした実装固有の情報にアクセスできるため利便性が高まります。


### 例外を投げない暗黙の変換が可能か調べる `is_nothrow_convertible` [(P0758R1)](https://wg21.link/P0758R1)
型 `From` から型 `To` への暗黙の変換が可能であるかを調べる型特性クラス `std::is_convertible<class From, class To>` が C++11 から導入されましたが、その変換が `noexcept` でもあるかを調べられるバージョンは実装されていませんでした。
このことが原因で、`std::decay_copy` の提案 ([N3255](http://wg21.link/n3255)) において、適切な `noexcept` 例外仕様を移植性のある方法で定義できない問題 ([LWG 2040](http://wg21.link/lwg2040)) が指摘されていました。
```C++
template <class T> 
typename decay<T>::type decay_copy(T&& v) noexcept(??? /* is_nothrow_convertible<T, T>::value */);
```
C++20 からは、`noexcept` な暗黙の変換が可能であることを調べる新しい型特性クラス `std::is_nothrow_convertible<class From, class To>` が実装されることで問題を解消できます。
既存の標準ライブラリ関数においても、`std::basic_string` のメンバ関数テンプレートに、より適切な `noexcept` 例外仕様を定義するために活用されます。

```C++
template <class T>
size_type find(const T& t, size_type pos = 0) const noexcept(is_nothrow_convertible_v<const T&, basic_string_view<CharT, Traits>>);
```

### ポインタライクなオブジェクトからアドレスを取得する `std::to_address()` 関数 [(P0653R2)](https://wg21.link/P0653R2)
ポインタライクなオブジェクトを引数にとり、それが表すのと同じアドレスを生ポインタで返す関数 `std::to_address(p)` が追加されます。オブジェクトがポインタ型の場合はその値を返し、それ以外の場合、`std::pointer_traits<Ptr>::to_address(p)` の特殊化が定義されていて使えればその戻り値を、そうでない場合は `std::to_address(p.operator->())` の戻り値を返します。


### `<complex>` ヘッダの関数の `constexpr` 対応を強化 [(P0415R1)](https://wg21.link/P0415R1)
`<complex>` ヘッダが提供する関数のうち、複素数の四則演算、ノルムの取得、共役複素数の取得など、`constexpr` 非対応の数学関数 (sqrt など) を使わずに実装できるものが `constexpr` 化されます。


### コンパイル時評価の文脈か実行時評価の文脈かを判別できる `std::is_constant_evaluated()` 関数 [(P0595R2)](https://wg21.link/P0595)
C++17 までは、実行するコードを、コンパイル時評価か実行時評価かに応じて使い分ける方法はありませんでした。C++20 では、コンパイル時評価されている文脈では `true` を、それ以外の場合では `false` を返す `std::is_constant_evaluated()` 関数が `<type_traits>` ヘッダに追加されます。例えば標準ライブラリで `constexpr` 対応していないような数学関数を提供する際、コンパイル時評価では `constexpr` 版の実装を、実行時には非 `constexpr` の標準ライブラリの実装を提供するよう選択させることができます。なお、`std::is_constant_evaluated()` を `if constexpr` の `( )` 内や `static_assert` 内で使うと常に `true` に評価されてしまうので注意が必要です。基本的には `if (std::is_constant_evaluated())` と書きます。

```cpp
#include <cmath>
#include <type_traits>
#include <iostream>
#include <iomanip>

constexpr float Sin_impl(float x2, int i, int k, float xn, long long nf)
{
	return (i > 10) ? 0.0f : (k * xn / nf + Sin_impl(x2, i + 2, -k, xn * x2, nf * (i + 1) * (i + 2)));
}

constexpr float Sin(float x)
{
	if (std::is_constant_evaluated())
	{
		return Sin_impl(x * x, 1, 1, x, 1);
	}
	else
	{
		return std::sin(x);
	}
}

int main()
{
	constexpr float Pi = 3.14159265f;
	constexpr float theta = Pi / 4.0;

	constexpr float x1 = Sin(theta); // コンパイル時計算	
	float x2 = Sin(theta);  // 実行時計算

	std::cout << std::setprecision(16);
	std::cout << x1 << '\n';
	std::cout << x2 << '\n';
}
```
```
0.7071068286895752
0.7071067690849304
```

### 浮動小数点数型のアトミック操作を拡張 [(P0020R6)](https://wg21.link/P0020R6)
`std::atomic<T>` の `float`, `double`, `long double` 型の特殊化に、メンバ関数 `fetch_add()`, `fetch_sub()`, `operator+=()`, `operator-=()` が追加されます。


### `std::memory_order` を `enum class` に変更 [(P0439R0)](https://wg21.link/P0439R0)
C++17 まで `enum` で定義されていた `std::memory_order` を、モダンな C++ 文法と型安全のために、`enum class` で定義する仕様に変更されます。これまでの表記は定数で提供されるようになるため、既存のソースコードは影響を受けません。また、バイナリ互換性のために、`enum class` の基底型の選択は実装に任されています。

```C++ tab="C++17"
namespace std
{
	typedef enum memory_order
	{
		memory_order_relaxed,
		memory_order_consume,
		memory_order_acquire,
		memory_order_release,
		memory_order_acq_rel,
		memory_order_seq_cst
	} memory_order;
}
```

```C++ tab="C++20"
namespace std
{
	enum class memory_order /* : unspecified */
	{
		relaxed,
		consume,
		acquire,
		release,
		acq_rel,
		seq_cst
	};

	inline constexpr memory_order memory_order_relaxed = memory_order::relaxed;
	inline constexpr memory_order memory_order_consume = memory_order::consume;
	inline constexpr memory_order memory_order_acquire = memory_order::acquire;
	inline constexpr memory_order memory_order_release = memory_order::release;
	inline constexpr memory_order memory_order_acq_rel = memory_order::acq_rel;
	inline constexpr memory_order memory_order_seq_cst = memory_order::seq_cst;
}
```
