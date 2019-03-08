# C++23 以降に向けた提案
議論が進行中のおもな提案です。標準化会議によって規格化が妥当でないと判断された場合は採択されません。

## 標準ライブラリ モジュール
- [Standard Library Modules (P0581)](https://wg21.link/p0581) 

`<iostream>` や `<vector>` など、すべての C++ 標準ライブラリヘッダをモジュールとして提供する提案です。言語機能としてのモジュールは C++20 に導入されましたが、標準ライブラリへの導入は議論が間に合わず、C++23 に先送りされました。標準ライブラリのモジュールへの移行に合わせて、[標準ライブラリを再編成する提案](https://wg21.link/p1453)も出されています。

## `operator[]` に複数の引数
- [Subscripts On Parade (P1277)](https://wg21.link/p1277)

`m[i, j, k]` のように、`operator[]` の引数に複数の値を渡せるようにする提案です。多次元配列の要素に簡潔な式でアクセスできるようになります。この書き方は、C++17 規格ではコンマ演算子と解釈されて `m[k]` になってしまうため、まずは[現行の文法を deprecated にする提案](https://wg21.link/p1161)が提出されています。`std::span` の多次元版である [`std::mdspan` の提案](https://github.com/kokkos/array_ref)での活用も期待されます。

## 低レベル オーディオ API
- [A Standard Audio API for C++:
Motivation, Scope, and Basic Design (P1386)](https://wg21.link/p1386)

音声再生・録音を行う低レベル API `std::audio` を追加する提案です。以前頓挫した 2D Graphics API の標準化に比べれば、枯れている技術なので実現可能性は低くはなさそうです。波形データをオーディオデバイスとやり取りすることが主要な役割で、音声コーデックやフィルタなどの機能は提供しません。[実験的な実装を公開](https://github.com/stdcpp-audio/libstdaudio)して、設計に関するフィードバックを集めている段階です。

## ネットワーク
- [Working Draft, C++ Extensions for Networking (N4734)](https://wg21.link/n4734)

ネットワーク機能を標準ライブラリに導入する提案です。[Boost.Asio](https://www.boost.org/doc/libs/release/doc/html/boost_asio.html) をベースに、ソケット通信、名前解決、インターネットプロトコル、タイマー、バッファ、非同期処理のための各種機能を実装する予定です。

## リフレクション
- [Working Draft, C++ Extensions for Reﬂection (N4766)](https://wg21.link/n4766)

C++ にリフレクションを導入する提案です。C# のような動的なリフレクションではなく、コンパイル時に情報を取得する静的なリフレクションです。

```C++
#include <experimental/reflect>

using namespace std::experimental::reflect;

constexpr void f(int arg1, int arg2);

using f_mobj = reflexpr(f(0, 1));

auto name = get_name_v<get_element_t<0, get_parameters_t<f_mobj>>>; // "arg1"

auto is_constexpr_function = is_constexpr<f_mobj>::value; // true
```

この機能を前提とした[メタクラスの提案](https://wg21.link/p0707)も議論が進んでいます。

## Executors



