description: C++ プログラムのコンパイル・実行ができる Web サイトの紹介

# オンラインコンパイラ

## 開発環境

C++ プログラムのコンパイル・実行ができる Web サイトです。

|                                                           | コンパイラ                       | 日本語入出力 | インタラクティブ | 複数ファイル | Share      |
| --------------------------------------------------------- | -------------------------------- | ------------ | ---------------- | ------------ | ---------- |
| [Wandbox](https://wandbox.org/)                           | gcc HEAD 9.0.1, Clang HEAD 9.0.0 | :fa-check:   |                  | :fa-check:   | :fa-check: |
| [repl.it](https://repl.it/languages/cpp)                  | Clang 7.0.0                      | :fa-check:   | :fa-check:       | :fa-check:   | :fa-check: |
| [paiza.io](https://paiza.io/ja/projects/new?language=cpp) | Clang 6.0.0                      | :fa-check:   |                  | :fa-check:   | :fa-check: |
| [GDB Online](https://www.onlinegdb.com/)                  | gcc 7.3.0                        | :fa-check:   | :fa-check:       | :fa-check:   | :fa-check: |
| [Ideone](https://ideone.com/)                             | gcc 6.3.0                        | :fa-check:   |                  |              | :fa-check: |
| [C++ Shell](http://cpp.sh/)                               | gcc 4.9.2                        |              | :fa-check:       |              | :fa-check: |

## C++ Insights: ソース → ソース変換
[C++ Insights](https://cppinsights.io/) は、ラムダ式、range-based for, 構造化束縛などで何が起こっているのかを、プログラムを単純なソースコードに分解することで可視化するオンラインのツールです。  

#### 入力例
```c++
#include <cstdio>

int main()
{
    const char arr[10]{2,4,6,8};

    for(const char& c : arr)
    {
      printf("c=%c\n", c);
    }
}
```
#### 出力
```c++
#include <cstdio>

int main()
{
  const char arr[10] = {2, 4, 6, 8, '\0', '\0', '\0', '\0', '\0', '\0'};
  {
    char const (&__range1)[10] = arr;
    const char * __begin1 = __range1;
    const char * __end1 = __range1 + 10l;
    
    for( ; __begin1 != __end1; ++__begin1 )
    {
      const char & c = *__begin1;
      printf("c=%c\n", static_cast<int>(c));
    }
  }
}
```

## Compiler Explorer: ソース → アセンブリ変換

### 概要
[Compiler Explorer](https://godbolt.org/) は、C, C++, Rust, Swift などのソースコードをコンパイルしてアセンブリを表示するオンラインコンパイラです。複数タブを使って、GCC, Clang, MSVC, ICC などのコンパイラや、コンパイルオプションを変えたときの結果を比較できます。

#### 入力例
```c++
int square(int num) {
    return num * num;
}
```

#### 出力
```asm
square(int):
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-4], edi
        mov     eax, DWORD PTR [rbp-4]
        imul    eax, DWORD PTR [rbp-4]
        pop     rbp
        ret
```

### URL からインクルード
Compiler Explorer には、Web 上のファイルを `#include "URL"` でインクルードできる独自拡張機能があります。この機能を使うと、GitHub などに公開されているシングルヘッダライブラリをプログラムの中で簡単に使えます（例: https://godbolt.org/z/OV-vGQ）


## Stensal: セグメンテーション違反の原因を表示
[Stensal](https://segfault.stensal.com/) は、C, C++ プログラムを実行して、セグメンテーション違反などメモリに関する問題が発生したときに、その箇所と原因をわかりやすく表示するオンラインコンパイラです。バッファオーバーラン、Null ポインタの参照外し、未初期化変数の利用などの問題を明らかにします。実装には Wandbox が使われています。

#### 入力例
```C++
#include <iostream>

int main()
{
	const char s[] = { 'A', 'B', 'C' };

	std::cout << s << '\n';
}
```

#### 出力例
```
=========== Start of #0 stensal runtime message ===========

  Runtime error: **[out-of-bounds read]**  
  Continuing execution can cause undefined behavior, abort!

 ```stensal-diagnostic-info
-
- Reading 4 bytes from 0x937c414 will read undefined values.
- 
- The memory-space-to-be-read (start:0x937c414, size:3 bytes) is bound to 's' at
-     file:/prog.cc::5, 0
- 
-  0x937c414               0x937c416
-  +------------------------------+
-  | the memory-space-to-be-read  |......
-  +------------------------------+
-  ^~~~~~~~~~
-  the read starts at the memory begin.
- 
- Stack trace (most recent call first) of the read.
- [1]  file:/musl-1.1.10/src/string/strlen.c::91, 3
- [2]  @[unknown_id 291]
- [3]  file:/prog.cc::7, 2
- [4]  [libc-start-main]
-
 ```
error code (56,213)

============ End of #0 stensal runtime message ============
```
