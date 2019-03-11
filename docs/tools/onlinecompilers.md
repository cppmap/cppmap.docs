# オンラインコンパイラ

## 開発環境

C++ プログラムのコンパイル・実行ができる Web サイトです。

|                                                           | コンパイラ                        | 日本語入出力 | インタラクティブ | 複数ファイル | Share     |
| --------------------------------------------------------- | -------------------------------- | ----------- | -------------- | ----------- | ---------- |
| [Wandbox](https://wandbox.org/)                           | gcc HEAD 9.0.1, Clang HEAD 9.0.0 | :fa-check:  |                | :fa-check:  | :fa-check: |
| [repl.it](https://repl.it/languages/cpp)                  | Clang 6.0.0                      | :fa-check:  | :fa-check:     | :fa-check:  | :fa-check: |
| [paiza.io](https://paiza.io/ja/projects/new?language=cpp) | Clang 6.0.0                      | :fa-check:  |                | :fa-check:  | :fa-check: |
| [Ideone](https://ideone.com/)                             | gcc 6.3.0                        | :fa-check:  |                |             | :fa-check: |
| [C++ Shell](http://cpp.sh/)                               | gcc 4.9.2                        |             | :fa-check:     |             | :fa-check: |

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
