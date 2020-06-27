description: C++ の並行・並列ライブラリの紹介

# 並行・並列

## マルチタスクスケジューリング

- [Cpp-Taskflow](https://github.com/cpp-taskflow/cpp-taskflow) <small>MIT</small>
    - C++17 以上でモダンな API でタスクフローが組める
    - work-stealing スケジューラによる高効率なタスクスケジューリング
    - `chrome://tracing` で実行状況をモニタできる
    - タスクフローをグラフ出力できる

## コルーチン

- [cppcoro](https://github.com/lewissbaker/cppcoro) <small>MIT</small>
    - C++20コルーチンの細かな制御/拡張ポイントを担当するライブラリ部分の実装
    - [C++20のコルーチン for アプリケーション - Qiita](https://qiita.com/Fuyutsubaki/items/a4c9921587ce53d95e55)
