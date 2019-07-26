description: C++ のテストフレームワークの紹介

# 単体テストフレームワーク

- [googletest](https://github.com/google/googletest) <small>BSD 3-Clause</small>
- [Catch](https://github.com/catchorg/Catch2) <small>BSL 1.0</small>
    - C++11 以上
    - シングルヘッダで外部依存ライブラリなし
    - 簡単に使い始められるインターフェース
- [doctest](https://github.com/onqtam/doctest) <small>MIT</small>
    - C++11 以上
    - シングルヘッダで外部依存ライブラリなし
    - ヘッダのコンパイル時間が非常に短く（[ベンチマーク](https://github.com/onqtam/doctest/blob/master/doc/markdown/benchmarks.md)）、テスト実行のイテレーションを速く回せるのが売り
